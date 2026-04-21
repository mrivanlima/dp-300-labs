## Last session: April 21, 2026

## Lab structure
- Folder per lab: labXX-topic-name
- Each folder contains one README.md
- Format: What I configured, T-SQL used, Key concepts, comparison tables, Exam tips
- Repo: https://github.com/mrivanlima/dp-300-labs

## Next session starts at
Domain 3 - DMVs (Dynamic Management Views)

## Key exam tips so far (add these)
- Query Store = plan history + runtime stats persisted inside the database
- Goes READ_ONLY silently when full — monitor actual_state_desc vs desired_state_desc
- Forced plans travel with backups — clear Query Store after restore to non-equivalent environments
- avg_duration in Query Store = microseconds
- Parameter sniffing = one plan compiled for first parameter value, reused for all
- High variation in Query Store = suspect parameter sniffing
- Regressed Queries report = where you force plans from the GUI
