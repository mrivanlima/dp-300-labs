# Lab 03 — Deploy Azure SQL Using PowerShell

## Commands used

# Read existing database
Get-AzSqlDatabase `
  -ResourceGroupName "dp300-labs-rg" `
  -ServerName "dp300-lab01-ivan-lima" `
  -DatabaseName "dp300-lab01"

# Create a new database
New-AzSqlDatabase `
  -ResourceGroupName "dp300-labs-rg" `
  -ServerName "dp300-lab01-ivan-lima" `
  -DatabaseName "dp300-lab03-ps" `
  -RequestedServiceObjectiveName "Basic" `
  -BackupStorageRedundancy "Local"

# Delete a database
Remove-AzSqlDatabase `
  -ResourceGroupName "dp300-labs-rg" `
  -ServerName "dp300-lab01-ivan-lima" `
  -DatabaseName "dp300-lab03-ps"

# List all databases on server
Get-AzSqlDatabase `
  -ResourceGroupName "dp300-labs-rg" `
  -ServerName "dp300-lab01-ivan-lima"

## CLI vs PowerShell cheat sheet
| Action | CLI                | PowerShell             |
|--------|--------------------|------------------------|
| Create | az sql db create   | New-AzSqlDatabase      |
| Read   | az sql db show     | Get-AzSqlDatabase      |
| Delete | az sql db delete   | Remove-AzSqlDatabase   |
| List   | az sql db list     | Get-AzSqlDatabase      |

## Key concepts learned
- PowerShell uses verb-noun pattern: Get, New, Remove
- master is a system database present on every server
- Default collation: SQL_Latin1_General_CP1_CI_AS
- CI = Case Insensitive, AS = Accent Sensitive
- NotFound error means resource already deleted

## Exam tip
Recognize New-AzSqlDatabase vs az sql db create —
same result, different tools. Both valid for automation.
