# Lab 08 — Query Store

## What I configured
Enabled and queried Query Store on an Azure SQL Database to monitor
query execution plans and runtime statistics.

## T-SQL used

-- Check Query Store status
SELECT desired_state_desc, actual_state_desc,
       current_storage_size_mb, max_storage_size_mb,
       query_capture_mode_desc
FROM sys.database_query_store_options;

-- Generate query activity
SELECT * FROM dbo.Customers WHERE CustomerID = 1;
SELECT * FROM dbo.Customers WHERE CustomerID = 2;
SELECT * FROM dbo.Customers WHERE CustomerID = 3;
GO 10

-- View raw Query Store data with time buckets
SELECT
    qt.query_sql_text,
    q.query_id,
    p.plan_id,
    rs.avg_duration,
    rs.count_executions
FROM sys.query_store_query_text qt
JOIN sys.query_store_query q ON qt.query_text_id = q.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
ORDER BY rs.avg_duration DESC;

-- View aggregated stats across all time buckets
SELECT
    qt.query_sql_text,
    q.query_id,
    p.plan_id,
    SUM(rs.count_executions) AS total_executions,
    AVG(rs.avg_duration) AS avg_duration_overall
FROM sys.query_store_query_text qt
JOIN sys.query_store_query q ON qt.query_text_id = q.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
GROUP BY qt.query_sql_text, q.query_id, p.plan_id
ORDER BY avg_duration_overall DESC;

-- Force a plan
EXEC sp_query_store_force_plan @query_id = 73, @plan_id = 3;

-- Unforce a plan
EXEC sp_query_store_unforce_plan @query_id = 73, @plan_id = 3;

-- Configure Query Store size and cleanup policy
ALTER DATABASE [YourDatabase] SET QUERY_STORE (
    MAX_STORAGE_SIZE_MB = 1024,
    CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30)
);

-- Clear Query Store after restore to non-equivalent environment
ALTER DATABASE [YourDatabase] SET QUERY_STORE CLEAR;

## Key concepts learned
- Query Store persists execution plans, query text, and runtime stats inside the database
- Data survives restarts and travels with backups/restores
- Stats are stored in time buckets — aggregate across buckets for true averages
- avg_duration is in microseconds, not milliseconds
- Query Store goes READ_ONLY silently when storage is full — monitor actual_state_desc
- Parameter sniffing: SQL Server normalizes literals into parameters (@1) and tracks all executions under one query_id
- Forced plans travel with the database on restore — always clear after restore to non-equivalent environments

## Query Store internal stores
| Store | What it holds |
|-------|--------------|
| Query Store | Normalized query text, hashed |
| Plan Store | Execution plans |
| Runtime Stats Store | Duration, CPU, reads, writes, row counts per time bucket |

## Query Store GUI reports
| Report | Purpose |
|--------|---------|
| Regressed Queries | Queries where performance degraded after a plan change |
| Overall Resource Consumption | Top resource consumers over time |
| Top Resource Consuming Queries | Highest CPU/duration/reads queries |
| Queries With Forced Plans | All queries with manually pinned plans |
| Queries With High Variation | High standard deviation in execution time — suspect parameter sniffing |
| Query Wait Statistics | Where queries spend time waiting (locks, IO, CPU) |
| Tracked Queries | Monitor specific query_ids over time |

## Exam tips
- Query Store = flight data recorder for SQL Server
- Goes READ_ONLY when full, not FIFO eviction — fails silently
- Forced plan = emergency workaround, not a fix — always resolve root cause
- Clear Query Store after restore to dev/test from production backup
- AUTO capture mode captures everything including internal Azure queries
- High variation in execution time = suspect parameter sniffing
