# Lab 02 — Deploy Azure SQL Using Azure CLI

## Commands used

# Show existing database
az sql db show \
  --resource-group dp300-labs-rg \
  --server dp300-lab01-ivan-lima \
  --name dp300-lab01 \
  --output table

# Create a new database
az sql db create \
  --resource-group dp300-labs-rg \
  --server dp300-lab01-ivan-lima \
  --name dp300-lab02-cli \
  --service-objective Basic \
  --output table

# Delete a database
az sql db delete \
  --resource-group dp300-labs-rg \
  --server dp300-lab01-ivan-lima \
  --name dp300-lab02-cli \
  --yes

## Key concepts learned
- Azure CLI uses `az sql` commands for SQL resources
- --service-objective sets the pricing tier
- --output table formats results cleanly
- Basic tier = 5 DTUs, 2GB max size
- DTU = blended measure of CPU, memory, and I/O

## Exam tip
For automated/repeatable deployments the exam expects
Azure CLI, PowerShell, or ARM/Bicep — never the portal.
