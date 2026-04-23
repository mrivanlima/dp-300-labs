# Lab 09 — Dynamic Management Views (DMVs)

## What I practiced
Queried key DMVs to monitor active sessions, running requests,
query performance, missing indexes, and wait statistics.

## T-SQL used

-- View all active requests with blocking info
SELECT 
    session_id,
    status,
    command,
    wait_type,
    wait_time,
    blocking_session_id,
    cpu_time,
    logical_reads,
    text.text AS query_text
FROM sys.dm_exec_requests
CROSS APPLY sys.dm_exec_sql_text(sql_handle) AS text
WHERE session_id > 50;

-- View all sessions including idle (LEFT JOIN to catch idle connections)
SELECT
    s.session_id,
    s.status,
    s.login_name,
    s.host_name,
    s.program_name,
    s.last_request_start_time,
    r.command,
    r.wait_type,
    r.blocking_session_id
FROM sys.dm_exec_sessions s
LEFT JOIN sys.dm_exec_requests r ON s.session_id = r.session_id
WHERE s.session_id > 50;

-- Top 10 slowest queries by average elapsed time
SELECT TOP 10
    total_elapsed_time / execution_count AS avg_elapsed_time,
    total_logical_reads / execution_count AS avg_logical_reads,
    total_worker_time / execution_count AS avg_cpu_time,
    execution_count,
    SUBSTRING(st.text, 1, 200) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
ORDER BY avg_elapsed_time DESC;

-- Missing indexes ranked by impact
SELECT TOP 10
    mid.statement AS table_name,
    migs.unique_compiles,
    migs.user_seeks,
    migs.user_scans,
    migs.avg_total_user_cost * migs.avg_user_impact * (migs.user_seeks + migs.user_scans) AS index_impact_score,
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns
FROM sys.dm_db_missing_index_group_stats migs
JOIN sys.dm_db_missing_index_groups mig ON migs.group_handle = mig.index_group_handle
JOIN sys.dm_db_missing_index_details mid ON mig.index_handle = mid.index_handle
ORDER BY index_impact_score DESC;

-- Wait statistics (filtered to remove benign background waits)
SELECT TOP 10
    wait_type,
    waiting_tasks_count,
    wait_time_ms,
    wait_time_ms / waiting_tasks_count AS avg_wait_ms,
    max_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE waiting_tasks_count > 0
    AND wait_type NOT IN (
        'SLEEP_TASK', 'BROKER_TO_FLUSH', 'BROKER_TASK_STOP',
        'CLR_AUTO_EVENT', 'DISPATCHER_QUEUE_SEMAPHORE',
        'FT_IFTS_SCHEDULER_IDLE_WAIT', 'HADR_WORK_QUEUE',
        'HADR_FILESTREAM_IOMGR_IOCOMPLETION',
        'HADR_FABRIC_CALLBACK', 'XE_DISPATCHER_WAIT',
        'XE_TIMER_EVENT', 'WAITFOR', 'BROKER_EVENTHANDLER',
        'CHECKPOINT_QUEUE', 'DBMIRROR_EVENTS_QUEUE',
        'SQLTRACE_BUFFER_FLUSH', 'SLEEP_DBSTARTUP',
        'SLEEP_DBTASK', 'SLEEP_TEMPDBSTARTUP'
    )
ORDER BY wait_time_ms DESC;

## Key concepts learned
- DMVs = live window into SQL Server internals, most reset on restart
- session_id > 50 filters out internal system sessions
- dm_exec_requests = active requests only
- dm_exec_sessions = all connections including idle — use LEFT JOIN with requests
- blocking_session_id = 0 means not blocked, non-zero points to the blocker
- Head blocker = session with blocking_session_id = 0 at top of blocking chain
- logical_reads matter more than physical reads for tuning — measures work regardless of cache
- dm_exec_query_stats times are in microseconds
- Missing index DMV gives column order hints — equality columns always go first
- Do NOT blindly create every missing index — write overhead must be weighed against read gains

## DMV reference
| DMV | Scope | Purpose |
|-----|-------|---------|
| sys.dm_exec_requests | Server | Active executing requests only |
| sys.dm_exec_sessions | Server | All connected sessions including idle |
| sys.dm_exec_query_stats | Server | Aggregated query performance since last restart |
| sys.dm_db_missing_index_details | Database | Missing index recommendations from optimizer |
| sys.dm_os_wait_stats | Server | Cumulative wait statistics since last restart |

## Critical wait types
| Wait Type | Means | Investigate |
|-----------|-------|-------------|
| LCK_M_X | Exclusive lock wait | Blocking chain, long transactions |
| PAGEIOLATCH_SH | Disk IO wait | Missing indexes, table scans |
| CXPACKET | Parallelism | Max degree of parallelism settings |
| RESOURCE_SEMAPHORE | Memory grant wait | Query memory pressure |
| ASYNC_NETWORK_IO | Client too slow | Application consuming results |

## Performance diagnosis chain
1. Check wait stats → identify what SQL Server is waiting on
2. Check dm_exec_requests → find blocked or waiting sessions
3. Check dm_exec_query_stats → find high logical reads queries
4. Check missing indexes → find what indexes are needed
5. Check execution plan → confirm the problem
6. Fix → validate improvement

## Exam tips
- DMVs reset on restart — Query Store does not
- session_id 1-50 = system sessions, always filter them out
- blocking_session_id = 0 means head blocker, not unblocked
- PAGEIOLATCH_SH = disk IO = missing index first suspect
- Logical reads = pages read from buffer cache = measure of query work
- Equality columns go before inequality columns in index design
- Missing index DMV has no awareness of write overhead — you must weigh the tradeoff
