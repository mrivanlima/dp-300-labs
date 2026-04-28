# Lab 10 — Extended Events

## What I practiced
Created an Extended Events session to capture slow queries
exceeding 1 second duration, read results from the ring buffer,
and monitored events using the Live Data viewer in SSMS.

## T-SQL used

-- Create Extended Events session with ring buffer target
CREATE EVENT SESSION [SlowQueryCapture] ON DATABASE
ADD EVENT sqlserver.sql_statement_completed
(
    ACTION
    (
        sqlserver.sql_text,
        sqlserver.client_app_name,
        sqlserver.username,
        sqlserver.session_id
    )
    WHERE duration > 1000000  -- microseconds, 1000000 = 1 second
)
ADD TARGET package0.ring_buffer
(SET max_memory = 51200)  -- 50MB
WITH
(
    MAX_DISPATCH_LATENCY = 5 SECONDS
);

-- Start the session
ALTER EVENT SESSION [SlowQueryCapture] ON DATABASE STATE = START;

-- Generate a slow query to capture
WAITFOR DELAY '00:00:02';
SELECT * FROM dbo.Customers;

-- Read raw XML from ring buffer
SELECT CAST(target_data AS XML) AS raw_xml
FROM sys.dm_xe_database_session_targets t
JOIN sys.dm_xe_database_sessions s ON t.event_session_address = s.address
WHERE s.name = 'SlowQueryCapture'
AND t.target_name = 'ring_buffer';

-- Parse ring buffer XML into readable format
DECLARE @xml XML;

SELECT @xml = CAST(target_data AS XML)
FROM sys.dm_xe_database_session_targets t
JOIN sys.dm_xe_database_sessions s ON t.event_session_address = s.address
WHERE s.name = 'SlowQueryCapture'
AND t.target_name = 'ring_buffer';

SELECT
    @xml.value('(/RingBufferTarget/event/@name)[1]', 'varchar(50)') AS event_name,
    @xml.value('(/RingBufferTarget/event/@timestamp)[1]', 'varchar(50)') AS event_timestamp,
    @xml.value('(/RingBufferTarget/event/data[@name="duration"]/value)[1]', 'bigint') AS duration_microseconds,
    @xml.value('(/RingBufferTarget/event/data[@name="duration"]/value)[1]', 'bigint') / 1000000 AS duration_seconds,
    @xml.value('(/RingBufferTarget/event/data[@name="statement"]/value)[1]', 'nvarchar(max)') AS statement,
    @xml.value('(/RingBufferTarget/event/action[@name="sql_text"]/value)[1]', 'nvarchar(max)') AS sql_text,
    @xml.value('(/RingBufferTarget/event/action[@name="username"]/value)[1]', 'nvarchar(50)') AS username;

-- Stop and drop the session
ALTER EVENT SESSION [SlowQueryCapture] ON DATABASE STATE = STOP;
DROP EVENT SESSION [SlowQueryCapture] ON DATABASE;

## Four core concepts
| Concept | Definition |
|---------|------------|
| Event | The trigger — something that happens inside SQL Server to capture (deadlock, slow query, failed login) |
| Target | Where captured data goes — ring buffer, event file, or event counter |
| Action | The payload — additional data collected when the event fires (sql_text, username, session_id) |
| Predicate | The filter — applied at source before data is collected, keeps overhead minimal |

## Targets comparison
| Target | Storage | Persists | Best for |
|--------|---------|----------|---------|
| Ring buffer | Memory | No — lost on restart | Real time troubleshooting |
| Event file | Disk | Yes | Intermittent problems, long term capture |
| Event counter | Memory | No | Counting event frequency only |

## Key concepts learned
- Extended Events replaced SQL Profiler — lighter, more precise, production safe
- SQL Profiler is deprecated — never use it in production
- Predicates filter at the source before data is collected — this is what keeps overhead low
- Ring buffer data is lost when session stops or server restarts
- Event file persists to disk — use for intermittent problems you cannot watch in real time
- statement = the specific slow statement, sql_text = the full batch context
- duration is always in microseconds — divide by 1000000 for seconds
- Live Data viewer in SSMS shows events in real time without XML parsing
- sys.dm_xe_database_sessions = database scoped DMV for XEvent sessions

## Extended Events vs SQL Profiler
| | SQL Profiler | Extended Events |
|-|-------------|-----------------|
| Status | Deprecated | Current standard |
| Overhead | High | Minimal |
| Filtering | After capture | At source (predicate) |
| Production safe | No | Yes |
| Flexibility | Low | High |

## Exam tips
- "Minimal performance impact + capture query events" = Extended Events
- Ring buffer = memory only, lost on restart
- Event file = disk, persists, use for intermittent issues
- Predicate = filter at source, not after — this is why XE is lightweight
- Four concepts: Event, Target, Action, Predicate
- duration in Extended Events = microseconds, same as DMVs and Query Store
- Watch Live Data in SSMS = real time event monitoring without T-SQL
