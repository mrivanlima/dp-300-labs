# dp-300-labs
# Lab 01 — Deploy Azure SQL Database

## What I built
- Resource group: dp300-labs-rg (East US 2)
- SQL Server: dp300-lab01-ivan-lima (West US)
- Database: dp300-lab01 (Basic tier)

## Key concepts learned
- Azure SQL Database is fully managed PaaS
- SQL Managed Instance is used for lift & shift from on-premises
- SQL on VMs gives full OS control (IaaS)
- Firewall rules control public network access
- Azure always runs on UTC time

## First query result
- DB_NAME(): dp300-lab01
- SQL Version: Microsoft SQL Azure 12.0.2000.8
- DateTime is UTC-based

## Exam tip
Use Managed Instance when the question mentions:
SQL Agent jobs, cross-database queries, or linked servers.
