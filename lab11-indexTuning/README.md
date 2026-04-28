# Lab 11 – Index Tuning

## Objective
Understand how indexes work in Azure SQL and SQL Server, identify fragmentation,
create covering indexes, and perform index maintenance operations.

## Concepts Covered
- Clustered vs nonclustered indexes
- Covering indexes and the INCLUDE clause
- Key Lookup and how to eliminate it
- Index fragmentation: REORGANIZE vs REBUILD
- Statistics and why they matter after maintenance

## Environment
- SQL Server 2017 (Docker local instance)
- Database: WideWorldImporters

## Step 1 – Check Index Fragmentation

```sql
USE WideWorldImporters;

SELECT 
    OBJECT_NAME(ips.object_id)                    AS table_name,
    i.name                                         AS index_name,
    ips.index_type_desc,
    ROUND(ips.avg_fragmentation_in_percent, 2)     AS fragmentation_pct,
    ips.page_count
FROM sys.dm_db_index_physical_stats(
    DB_ID(), NULL, NULL, NULL, 'LIMITED') AS ips
INNER JOIN sys.indexes i 
    ON ips.object_id = i.object_id 
    AND ips.index_id = i.index_id
WHERE ips.page_count > 100
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

**Key finding:** `PK_Sales_Invoices` (clustered index) at 85.57% fragmentation
across 11,357 pages. A fragmented clustered index impacts the entire table
because the clustered index is the table.

## Step 2 – REBUILD a Highly Fragmented Clustered Index

Use REBUILD when fragmentation is above 30%. Be aware that REBUILD is offline
by default on non-Enterprise editions.

```sql
ALTER INDEX PK_Sales_Invoices 
ON Sales.Invoices 
REBUILD;
```

**Result:** Fragmentation dropped from 85.57% to 0.22%. Page count reduced
from 11,357 to 7,835 — compacted and physically reordered on disk.

## Step 3 – REORGANIZE a Nonclustered Index

Use REORGANIZE when fragmentation is between 5-30%, or when you cannot take
the index offline. REORGANIZE is always online and safe during business hours.

```sql
ALTER INDEX IX_Sales_OrderLines_Perf_20160301_02
ON Sales.OrderLines
REORGANIZE;
```

**Important:** REORGANIZE does not update statistics automatically.
Always follow with UPDATE STATISTICS.

```sql
UPDATE STATISTICS Sales.OrderLines 
    IX_Sales_OrderLines_Perf_20160301_02;
```

**Result:** Fragmentation dropped from 98.79% to 0.46%. Page count reduced
from 1,240 to 865.

## Step 4 – Identify a Key Lookup Problem

Run this query with Actual Execution Plan enabled:

```sql
SELECT 
    o.OrderID,
    o.CustomerID,
    o.OrderDate,
    o.ContactPersonID
FROM Sales.Orders o
WHERE o.CustomerID = 1034
AND o.OrderDate >= '2015-01-01';
```

**Observation:** Execution plan shows Index Seek + Key Lookup + Nested Loops.
Key Lookup costs 98% of the query. SQL Server uses `FK_Sales_Orders_CustomerID`
to find rows but must go back to the clustered index for `OrderDate` and
`ContactPersonID`.

## Step 5 – Create a Covering Index to Eliminate the Key Lookup

Rules:
- Equality columns go first in the key
- Inequality columns go last in the key  
- Columns only needed for SELECT go in INCLUDE

```sql
CREATE NONCLUSTERED INDEX IX_Sales_Orders_CustomerID_OrderDate
ON Sales.Orders (CustomerID, OrderDate)
INCLUDE (ContactPersonID);
```

Re-run the SELECT query with Actual Execution Plan enabled.

**Result:** Single Index Seek. No Key Lookup. No Nested Loops. Query fully
covered by the index. SSMS reports "No issues found."

## Maintenance Decision Reference

| Fragmentation | Operation | Notes |
|---------------|-----------|-------|
| < 5% | None | Not worth the overhead |
| 5% – 30% | REORGANIZE | Online, safe during business hours |
| > 30% | REBUILD | Offline on Standard, online on Enterprise |

## Key Exam Tips
- Clustered index = the table. One per table. Fragmentation impacts everything.
- Nonclustered index = separate structure pointing back to clustered index key
- Key Lookup = expensive. Eliminate with covering indexes using INCLUDE
- REBUILD updates statistics automatically. REORGANIZE does not.
- Heap = table with no clustered index. Uses RID Lookup instead of Key Lookup.
- Equality columns first, inequality columns last in composite index keys
- sys.dm_db_index_physical_stats = check fragmentation
- sys.dm_db_missing_index_details = find indexes SQL Server recommends
