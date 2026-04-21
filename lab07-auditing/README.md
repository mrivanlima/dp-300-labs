# Lab 07 — Azure SQL Auditing

## What I configured
- Enabled database-level auditing on dp300-lab01
- Audit destination: Azure Storage Account (dp300auditivan)
- Retention: 90 days
- Authentication: Storage Access Keys

## Key concepts learned
- Auditing tracks who did what and when
- Two levels: Server level and Database level
- Server level overrides database level settings
- If server auditing is ON it applies to ALL databases

## Audit log destinations
| Destination | Best for |
|-------------|----------|
| Storage Account | Long term cheap retention |
| Log Analytics | Querying and dashboards |
| Event Hub | Streaming to external SIEM |

## Provider registration
-- Required before enabling auditing
az provider register --namespace Microsoft.Insights

-- Check registration status
az provider show -n Microsoft.Insights --query registrationState

## Retention days
- 0 = keep forever
- 90 = common compliance requirement
- Max = 999 days

## Exam tips
- "Cost effective long term retention" = Storage Account
- "Query and analyze logs" = Log Analytics
- "Stream to Splunk/external SIEM" = Event Hub
- Server level auditing always wins over database level
- Auditing required for HIPAA, PCI-DSS, SOX compliance
