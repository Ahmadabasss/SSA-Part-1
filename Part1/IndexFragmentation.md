# Index Fragmentation

## What is Index Fragmentation?

Index fragmentation occurs when the logical ordering of pages in an index does not match the physical ordering on disk. This can lead to inefficient data retrieval and degraded performance in database operations.

## Types of Index Fragmentation

1. **Internal Fragmentation**:
   - Occurs when there is free space within the index pages.
   - Example: If an index page has a capacity of 100 rows but contains only 70 rows, the remaining space is wasted and considered internal fragmentation.

2. **External Fragmentation**:
   - Happens when the logical order of index pages does not match the physical order.
   - Example: If an index is logically ordered as pages 1, 2, 3, 4, but physically stored as pages 1, 4, 2, 3, the access paths become inefficient.

## Causes of Index Fragmentation

- **Insertions, Updates, and Deletions**:
  - Frequent insertions, updates, and deletions can lead to page splits and cause fragmentation.
- **Page Splits**:
  - When a page becomes full and a new row is inserted, the page splits into two, leaving gaps and causing fragmentation.

## Identifying Index Fragmentation

You can identify index fragmentation using SQL queries. For example, in SQL Server, you can use the `sys.dm_db_index_physical_stats` dynamic management view (DMV):


## Example Script for Reorganizing or Rebuilding Indexes

Here is a script to identify fragmented indexes and automatically reorganize or rebuild them based on their fragmentation percentage:

```sql
DECLARE @TableName NVARCHAR(128)
DECLARE @IndexName NVARCHAR(128)
DECLARE @SchemaName NVARCHAR(128)
DECLARE @Fragmentation DECIMAL(5, 2)
DECLARE @Command NVARCHAR(MAX)

DECLARE index_cursor CURSOR FOR
SELECT
    dbschemas.name AS 'Schema',
    dbtables.name AS 'Table',
    dbindexes.name AS 'Index',
    indexstats.avg_fragmentation_in_percent
FROM
    sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') AS indexstats
    INNER JOIN sys.tables dbtables ON indexstats.object_id = dbtables.object_id
    INNER JOIN sys.schemas dbschemas ON dbtables.schema_id = dbschemas.schema_id
    INNER JOIN sys.indexes AS dbindexes ON indexstats.object_id = dbindexes.object_id
        AND indexstats.index_id = dbindexes.index_id
WHERE
    indexstats.database_id = DB_ID()
    AND indexstats.index_id > 0
ORDER BY
    indexstats.avg_fragmentation_in_percent DESC;

OPEN index_cursor
FETCH NEXT FROM index_cursor INTO @SchemaName, @TableName, @IndexName, @Fragmentation

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @Fragmentation > 30
        SET @Command = 'ALTER INDEX [' + @IndexName + '] ON [' + @SchemaName + '].[' + @TableName + '] REBUILD;'
    ELSE IF @Fragmentation > 10
        SET @Command = 'ALTER INDEX [' + @IndexName + '] ON [' + @SchemaName + '].[' + @TableName + '] REORGANIZE;'
    ELSE
        SET @Command = ''

    IF @Command <> ''
    BEGIN
        PRINT 'Executing: ' + @Command
        EXEC sp_executesql @Command
    END

    FETCH NEXT FROM index_cursor INTO @SchemaName, @TableName, @IndexName, @Fragmentation
END

CLOSE index_cursor
DEALLOCATE index_cursor


