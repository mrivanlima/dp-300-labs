# Lab 04 — Configure Microsoft Entra ID Authentication

## What I configured
- Set Microsoft Entra admin on SQL Server: dp300-lab01-ivan-lima
- Admin account: mrivanlima@gmail.com (guest/EXT user)
- Connected to database using Entra ID — no password required

## Key concepts learned
- Entra ID auth is configured at the SERVER level, not database level
- Only ONE Entra admin allowed per server (user or group)
- Best practice: use a GROUP as Entra admin, not individual user
- "Entra-only authentication" disables SQL auth completely
- #EXT# suffix means guest user from external tenant
- Managed Identity lets SQL Server authenticate to Key Vault
  without storing credentials in code

## Authentication comparison
| Method | Password stored? | Best for |
|--------|-----------------|----------|
| SQL auth | Yes (in DB) | Legacy, quick setup |
| Entra ID | No | Production, corporate |
| Entra-only | No | Most secure, disables SQL auth |

## Exam tips
- "Eliminate passwords" = Entra ID authentication
- "Most secure" = Entra-only authentication
- "Lift & shift needs SQL Agent" = Managed Instance
- TDE is ON by default for Azure SQL Database
