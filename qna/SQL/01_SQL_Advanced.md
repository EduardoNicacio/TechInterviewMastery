# SQL – Interview Questions & Answers

**Question**: What is the difference between INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN, and CROSS JOIN?

**Answer**: INNER JOIN returns only matching rows from both tables. LEFT JOIN returns all rows from the left table and matching rows from the right (NULL where no match). RIGHT JOIN is the opposite. FULL OUTER JOIN returns all rows from both tables with NULLs for non-matches. CROSS JOIN returns the Cartesian product (every combination of rows).

```sql
-- INNER JOIN
SELECT e.Name, d.DeptName FROM Employees e INNER JOIN Departments d ON e.DeptId = d.Id;
-- LEFT JOIN
SELECT e.Name, d.DeptName FROM Employees e LEFT JOIN Departments d ON e.DeptId = d.Id;
-- CROSS JOIN
SELECT e.Name, d.DeptName FROM Employees e CROSS JOIN Departments d;
```

---

**Question**: What is a SELF JOIN and when would you use it?

**Answer**: A SELF JOIN joins a table to itself, typically using different aliases. It is used for hierarchical data (e.g., employees and managers), finding duplicates, or comparing rows within the same table.

```sql
SELECT e1.Name AS Employee, e2.Name AS Manager
FROM Employees e1 LEFT JOIN Employees e2 ON e1.ManagerId = e2.Id;
```

---

**Question**: What are the differences between Primary Key, Foreign Key, and Unique Key?

**Answer**: A Primary Key uniquely identifies each row, enforces entity integrity, and is non-NULLable (only one per table). A Foreign Key maintains referential integrity by linking to a Primary Key in another table. A Unique Key ensures uniqueness but allows NULLs, and multiple can exist per table.

---

**Question**: What is the difference between Clustered and Non-Clustered Index?

**Answer**: A Clustered Index determines the physical storage order of data in a table (leaf nodes = actual data rows). A table can have only one. A Non-Clustered Index stores a separate structure with pointers to the data rows, and a table can have many (up to 999 in SQL Server).

```sql
-- Clustered index on Id
CREATE CLUSTERED INDEX IX_Employees_Id ON Employees(Id);
-- Non-clustered index on Name
CREATE NONCLUSTERED INDEX IX_Employees_Name ON Employees(Name);
```

---

**Question**: How does column order matter in a composite index?

**Answer**: Column order determines the sort order of the index B-tree. The leading column should be the most selective (highest cardinality) or the one used most frequently in equality predicates. The index supports searches on the leading column(s); queries filtering only on non-leading columns cannot use the index efficiently.

```sql
-- Good: queries filtering on DeptId or (DeptId, Status) can use this
CREATE INDEX IX_Orders_DeptId_Status ON Orders(DeptId, Status);
-- Poor: filtering only on Status cannot use the index efficiently
CREATE INDEX IX_Orders_Status_DeptId ON Orders(Status, DeptId);
```

---

**Question**: What are covering indexes and included columns?

**Answer**: A covering index contains all columns needed by a query, eliminating lookups into the base table. Included columns (INCLUDE clause) are stored only at the leaf level and do not affect index key order, allowing you to cover more queries without widening the key.

```sql
CREATE NONCLUSTERED INDEX IX_Orders_Covering ON Orders(OrderDate)
INCLUDE (CustomerName, TotalAmount);
-- Queries on OrderDate that also need CustomerName and TotalAmount are fully covered.
```

---

**Question**: What is the difference between Index Seek, Index Scan, and Table Scan?

**Answer**: An Index Seek uses the B-tree structure to locate rows directly (highly efficient for selective queries). An Index Scan reads all or most leaf pages of a non-clustered index (less efficient but can be preferred for large result sets). A Table Scan reads every row from the heap table (worst performance, used when no useful index exists).

---

**Question**: How do you analyze an Execution Plan?

**Answer**: Look for operators with high estimated cost percentage, especially table/scans, key lookups, sorts, and spools. Identify missing index hints, operator cost imbalances (e.g., 90% on one node), and large row estimates vs actuals (cardinality estimation errors). Use SET SHOWPLAN_XML or the GUI plan viewer.

```sql
SET STATISTICS IO ON; SET STATISTICS TIME ON;
SELECT * FROM Orders WHERE OrderDate = '2024-01-01';
```

---

**Question**: What are the ACID properties in databases?

**Answer**: Atomicity (all or nothing), Consistency (transactions preserve database rules), Isolation (concurrent transactions don't interfere), Durability (committed changes survive failures). These guarantee reliable transaction processing, typically enforced by the database engine via locking and logging.

---

**Question**: What are the SQL Server isolation levels?

**Answer**: READ UNCOMMITTED (allows dirty reads, no locks). READ COMMITTED (default, prevents dirty reads but allows non-repeatable reads and phantoms). REPEATABLE READ (holds shared locks until end of transaction, preventing non-repeatable reads). SERIALIZABLE (range locks, prevents phantoms). SNAPSHOT (uses row versioning, no locks but tempdb overhead).

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRAN;
SELECT * FROM Orders WHERE Status = 'Active';
-- Other transactions cannot insert/update rows matching this predicate
COMMIT;
```

---

**Question**: What are dirty reads, non-repeatable reads, and phantom reads?

**Answer**: A dirty read occurs when a transaction reads uncommitted data from another transaction. A non-repeatable read happens when a row value changes between two reads within the same transaction. A phantom read occurs when new rows matching a search condition appear between two reads within the same transaction.

---

**Question**: What causes deadlocks and how do you detect/prevent them?

**Answer**: Deadlocks occur when two transactions each hold locks the other needs, creating a circular wait. Detect them via SQL Server's system_health session or trace flag 1222. Prevent by accessing resources in a consistent order, keeping transactions short, using lower isolation levels, and considering indexed views.

```sql
-- Enable deadlock graph in system_health (default trace)
-- Or use trace flag 1222 to log deadlock info to error log
DBCC TRACEON(1222, -1);
```

---

**Question**: What does the NOLOCK hint do and what are its risks?

**Answer**: NOLOCK (equivalent to READ UNCOMMITTED) issues no shared locks and reads uncommitted data. It avoids blocking but risks dirty reads, non-repeatable reads, phantom reads, and data corruption if page splits occur mid-read. Use only for approximate reporting on read-only replicas.

```sql
SELECT COUNT(*) FROM Orders WITH (NOLOCK) WHERE Status = 'Pending';
```

---

**Question**: How does the Transaction Log and Write-Ahead Logging work?

**Answer**: Write-Ahead Logging (WAL) ensures log records are written to disk before data pages are modified. The transaction log records every change sequentially. On a crash, SQL Server replays committed transactions (redo) and rolls back uncommitted ones (undo) during recovery, guaranteeing durability and atomicity.

---

**Question**: What are the normal forms (1NF, 2NF, 3NF, BCNF)?

**Answer**: 1NF: atomic columns, no repeating groups. 2NF: 1NF + every non-key column fully dependent on the whole primary key. 3NF: 2NF + no transitive dependencies (non-key columns depend only on the key). BCNF: 3NF + every determinant must be a candidate key, handling overlapping composite key anomalies.

---

**Question**: When would you denormalize a database?

**Answer**: Denormalization (adding redundant data) improves read performance for reporting, analytics, or high-read/low-write workloads. Use it to avoid expensive joins, reduce index overhead, or meet latency SLAs. Always balance against write complexity and data consistency costs.

---

**Question**: What is the difference between CTE, Subquery, Temp Table, and Table Variable?

**Answer**: A CTE (Common Table Expression) is a named temporary result set within a single query's scope (no statistics, no index). A subquery is nested inside a query. A Temp Table (#temp) is stored in tempdb, supports indexes and statistics, and persists across batches. A Table Variable (@table) is scoped to a batch, has limited statistics and no indexes (unless declared with inline index).

```sql
-- CTE
WITH HighValueOrders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY Total DESC) AS rn
    FROM Orders WHERE Total > 1000
)
SELECT * FROM HighValueOrders WHERE rn <= 5;

-- Temp Table
SELECT * INTO #TempOrders FROM Orders WHERE Total > 1000;
CREATE INDEX IX_Temp ON #TempOrders(CustomerId);
```

---

**Question**: How does a Recursive CTE work?

**Answer**: A Recursive CTE has an anchor member (base result) and a recursive member that references the CTE by name. It iterates until the recursive step returns zero rows. Useful for tree traversal, org charts, or bill-of-materials queries.

```sql
WITH OrgChart AS (
    SELECT Id, Name, ManagerId, 0 AS Level FROM Employees WHERE ManagerId IS NULL
    UNION ALL
    SELECT e.Id, e.Name, e.ManagerId, Level + 1
    FROM Employees e INNER JOIN OrgChart o ON e.ManagerId = o.Id
)
SELECT * FROM OrgChart;
```

---

**Question**: What is the difference between ROW_NUMBER, RANK, DENSE_RANK, and NTILE?

**Answer**: ROW_NUMBER assigns a unique sequential number per partition. RANK gives the same rank to ties but skips subsequent numbers. DENSE_RANK gives the same rank to ties without skipping. NTILE divides rows into a specified number of approximately equal groups.

```sql
SELECT Name, Salary,
    ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RowNum,
    RANK()       OVER (ORDER BY Salary DESC) AS Rank,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRank,
    NTILE(4)     OVER (ORDER BY Salary DESC) AS Quartile
FROM Employees;
```

---

**Question**: How do aggregate window functions with OVER/PARTITION BY work?

**Answer**: They compute aggregates across a window (partition) of rows without collapsing the result set, unlike GROUP BY. Each row retains its identity while also showing aggregated values for its group.

```sql
SELECT OrderId, CustomerId, Total,
    SUM(Total)   OVER (PARTITION BY CustomerId) AS CustomerTotal,
    AVG(Total)   OVER (PARTITION BY CustomerId) AS CustomerAvg,
    COUNT(*)     OVER (PARTITION BY CustomerId) AS CustomerOrderCount
FROM Orders;
```

---

**Question**: What are LAG and LEAD window functions used for?

**Answer**: LAG accesses a previous row's value within a partition without a self-join. LEAD accesses a subsequent row's value. They are used for comparisons between current and prior/next rows, such as calculating deltas or trends.

```sql
SELECT OrderDate, Total,
    LAG(Total, 1, 0)  OVER (ORDER BY OrderDate) AS PreviousTotal,
    LEAD(Total, 1, 0) OVER (ORDER BY OrderDate) AS NextTotal,
    Total - LAG(Total, 1, 0) OVER (ORDER BY OrderDate) AS DayOverDayChange
FROM Orders;
```

---

**Question**: What is the difference between Views and Materialized Views?

**Answer**: A View is a virtual table (saved query) that runs each time it's queried, always showing current data with no storage overhead. A Materialized View (Indexed View in SQL Server) persists the result set physically on disk, providing faster reads but requiring maintenance and consuming storage.

```sql
-- View
CREATE VIEW ActiveCustomers AS SELECT * FROM Customers WHERE IsActive = 1;
-- Materialized / Indexed View (SQL Server)
CREATE VIEW SalesSummary WITH SCHEMABINDING AS
SELECT ProductId, SUM(Quantity) AS TotalQty, COUNT_BIG(*) AS Cnt
FROM Sales GROUP BY ProductId;
CREATE UNIQUE CLUSTERED INDEX IX_SalesSummary ON SalesSummary(ProductId);
```

---

**Question**: What is the difference between Stored Procedures, Functions, and Triggers?

**Answer**: Stored Procedures can have input/output parameters, modify data, and use transaction control. Functions return a scalar or table value, cannot modify data (in SQL Server), and are used inside queries. Triggers execute automatically on DML/DDL events, enabling auditing or cascading logic.

```sql
-- Stored Procedure
CREATE PROC sp_GetOrders @CustomerId INT AS
SELECT * FROM Orders WHERE CustomerId = @CustomerId;

-- Scalar Function
CREATE FUNCTION fn_TotalOrders(@CustomerId INT) RETURNS MONEY AS
BEGIN RETURN (SELECT SUM(Total) FROM Orders WHERE CustomerId = @CustomerId); END;

-- Trigger
CREATE TRIGGER trg_AuditDelete ON Employees AFTER DELETE AS
INSERT INTO AuditLog(TableName, Action, OldData) SELECT 'Employees', 'DELETE', deleted.Name FROM deleted;
```

---

**Question**: When should you use Cursors and what is their performance impact?

**Answer**: Cursors process rows one at a time in a loop, causing high overhead from round-trips and row-by-row operations. Avoid them in set-based SQL. Use them only when imperative row-by-row logic is unavoidable (e.g., complex calculations per row, calling external APIs). Prefer set-based operations, window functions, or recursive CTEs instead.

```sql
DECLARE cur CURSOR FOR SELECT Id FROM Orders WHERE Status = 'Pending';
OPEN cur; FETCH NEXT FROM cur INTO @Id;
WHILE @@FETCH_STATUS = 0 BEGIN
    EXEC sp_ProcessOrder @Id;
    FETCH NEXT FROM cur INTO @Id;
END
CLOSE cur; DEALLOCATE cur;
```

---

**Question**: What is the difference between UNION, UNION ALL, EXCEPT, and INTERSECT?

**Answer**: UNION combines two result sets and removes duplicates. UNION ALL combines without dedup (faster). EXCEPT returns rows from the first query not in the second. INTERSECT returns rows common to both queries. All require matching column counts and compatible data types.

```sql
SELECT City FROM Customers
INTERSECT
SELECT City FROM Suppliers;
-- Returns cities that have both customers and suppliers
```

---

**Question**: When should you use EXISTS vs IN vs JOIN for filtering?

**Answer**: EXISTS short-circuits on the first match and is best for semi-joins (checking existence). IN is cleaner for small lists or subqueries returning few values. JOIN can be more flexible but may multiply rows if the join produces duplicates. In modern optimizers, EXISTS and IN often produce similar plans; EXISTS is preferred for correlated subqueries.

```sql
-- EXISTS (best for correlated subqueries)
SELECT * FROM Customers c WHERE EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerId = c.Id);
-- IN (cleaner for static lists or simple subqueries)
SELECT * FROM Customers WHERE Id IN (SELECT CustomerId FROM Orders);
```

---

**Question**: What are SARGable queries and predicate pushdown?

**Answer**: A SARGable (Search ARGument ABLE) query allows the optimizer to use an index seek. Avoid wrapping indexed columns in functions (e.g., WHERE YEAR(OrderDate) = 2024) because the optimizer cannot use the index directly. Predicate pushdown means the query engine filters data as early as possible (at the storage engine level), reducing data movement.

```sql
-- SARGable
SELECT * FROM Orders WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
-- NOT SARGable (function on column prevents index seek)
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2024;
```

---

**Question**: What is parameter sniffing and how does it affect performance?

**Answer**: Parameter sniffing causes the optimizer to generate a plan based on the first parameter value encountered. That plan may be optimal for that value but suboptimal for others with different data distributions. Mitigate with OPTION (RECOMPILE), OPTIMIZE FOR UNKNOWN, or query hints using local variables/parameterization.

```sql
CREATE PROC sp_GetOrders @Status VARCHAR(20) AS
SELECT * FROM Orders WHERE Status = @Status
OPTION (OPTIMIZE FOR (@Status = 'Active')); -- Forces plan for 'Active'
```

---

**Question**: How do you prevent SQL injection?

**Answer**: Always use parameterized queries or stored procedures instead of dynamic SQL concatenation. Use ORM frameworks (EF Core, Dapper) that parameterize automatically. Avoid EXEC with concatenated user input. Validate and sanitize input as a defense-in-depth layer.

```sql
-- Vulnerable
EXEC('SELECT * FROM Users WHERE Name = ''' + @UserName + '''');
-- Safe
SELECT * FROM Users WHERE Name = @UserName;
```

---

**Question**: What are database sharding strategies?

**Answer**: Sharding horizontally partitions data across multiple databases (shards) based on a shard key. Common strategies: Range-Based (e.g., customers A–M on shard 1, N–Z on shard 2), Hash-Based (consistent hashing for even distribution), and Directory-Based (lookup service maps keys to shards). Sharding complicates joins, transactions, and rebalancing.

---

**Question**: What is the difference between horizontal and vertical partitioning?

**Answer**: Horizontal partitioning splits rows across tables/partitions based on a partition key (e.g., date ranges). Vertical partitioning splits columns into separate tables (e.g., moving infrequently used BLOB columns to another table). Horizontal improves query performance on subsets; vertical improves I/O for wide tables.

```sql
-- Horizontal partitioning (SQL Server)
CREATE PARTITION FUNCTION pf_OrderDate(DATETIME) AS RANGE RIGHT FOR VALUES ('2023-01-01', '2024-01-01');
CREATE PARTITION SCHEME ps_OrderDate AS PARTITION pf_OrderDate ALL TO ([PRIMARY]);
CREATE TABLE Orders (Id INT, OrderDate DATETIME) ON ps_OrderDate(OrderDate);
```

---

**Question**: What are the types of database replication?

**Answer**: Transactional Replication continuously replicates changes from publisher to subscriber (low latency, high consistency, used for reporting/DR). Merge Replication allows both publisher and subscriber to make changes (peer-to-peer, conflict resolution). Snapshot Replication takes a point-in-time copy periodically (simple, used for reference data).

---

**Question**: What are the backup and recovery strategies for SQL Server?

**Answer**: Full Backup captures the entire database. Differential Backup captures changes since the last full backup (faster, smaller). Transaction Log Backup captures log records (required for point-in-time recovery). Recovery strategy: restore full, then latest differential, then all subsequent log backups. RPO and RTO determine frequency.

```sql
-- Full backup
BACKUP DATABASE MyDB TO DISK = 'C:\Backup\MyDB_Full.bak';
-- Differential backup
BACKUP DATABASE MyDB TO DISK = 'C:\Backup\MyDB_Diff.bak' WITH DIFFERENTIAL;
-- Transaction log backup
BACKUP LOG MyDB TO DISK = 'C:\Backup\MyDB_Log.trn';
```

---

**Question**: What are the trade-offs between SQL and NoSQL databases?

**Answer**: SQL databases provide ACID transactions, strong consistency, and complex joins with a fixed schema. NoSQL databases (document, key-value, graph) offer horizontal scalability, flexible schema, and high write throughput but often sacrifice strong consistency and complex querying. Choose based on data structure, consistency needs, and scalability requirements.

---

**Question**: What is the CAP theorem and how does it apply to databases?

**Answer**: CAP theorem states a distributed data system can guarantee at most two of: Consistency (all nodes see the same data), Availability (every request gets a response), and Partition Tolerance (system works despite network splits). SQL databases typically favor CA or CP; NoSQL often favors AP.

---

**Question**: What are Redis use cases and data structures?

**Answer**: Redis is an in-memory key-value store used for caching, session storage, rate limiting, pub/sub, and real-time leaderboards. Data structures: Strings, Lists (queues), Sets (unique items), Sorted Sets (leaderboards), Hashes (objects), HyperLogLogs (cardinality estimation), and Streams (event logs).

```redis
# Leaderboard using Sorted Set
ZADD leaderboard 100 "user1" 200 "user2" 150 "user3"
ZREVRANGE leaderboard 0 2 WITHSCORES
# Cache with TTL
SET session:abc123 "{userId: 42}" EX 3600
```

---

**Question**: What is index fragmentation and how do you maintain indexes (REBUILD vs REORGANIZE)?

**Answer**: Fragmentation occurs when index pages become logically out of order due to inserts/updates/deletes. REORGANIZE (online, low resource) defragments the leaf level when fragmentation is below 30%. REBUILD (offline in some editions, more thorough) drops and recreates the index, recommended for fragmentation above 30%.

```sql
ALTER INDEX IX_Orders_OrderDate ON Orders REORGANIZE;
ALTER INDEX IX_Orders_OrderDate ON Orders REBUILD;
-- Check fragmentation
SELECT * FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('Orders'), NULL, NULL, 'LIMITED');
```

---

**Question**: What are query hints and when should you avoid them?

**Answer**: Query hints force specific optimizer behavior (e.g., LOOP JOIN, RECOMPILE, FORCE ORDER). Avoid them in general because they prevent the optimizer from adapting to changing data distributions and schema changes. Use sparingly as a last resort after analyzing execution plans and updating statistics.

```sql
SELECT * FROM Orders o INNER LOOP JOIN Customers c ON o.CustomerId = c.Id
OPTION (RECOMPILE, MAXDOP 1);
```

---

**Question**: How do you optimize TempDB performance?

**Answer**: Use multiple TempDB data files (equal size, one per CPU core up to 8), enable TF 1117 (auto-grow all files equally) and TF 1118 (uniform extent allocation). Place TempDB on fast SSDs. Minimize sorting/hashing operations by optimizing queries and indexes. Monitor PAGELATCH contention via sys.dm_os_wait_stats.

---

**Question**: What is lock escalation and what is its impact?

**Answer**: Lock escalation converts many fine-grained locks (row/page) into a single table lock when a threshold (5000 locks) is exceeded. This reduces memory overhead but increases blocking for concurrent transactions. Mitigate by partitioning tables, using READ COMMITTED SNAPSHOT, or breaking large batch operations into smaller transactions.

---

**Question**: How do you implement pagination efficiently (OFFSET/FETCH vs Keyset Pagination)?

**Answer**: OFFSET/FETCH skips rows that must still be read (performance degrades with depth). Keyset pagination (seeks based on last seen value) is O(1) per page regardless of depth and is preferred for large datasets.

```sql
-- OFFSET/FETCH (performance degrades with large offset)
SELECT Id, Name FROM Employees ORDER BY Id OFFSET 100 ROWS FETCH NEXT 20 ROWS ONLY;
-- Keyset pagination (stable, efficient)
SELECT Id, Name FROM Employees WHERE Id > 100 ORDER BY Id FETCH NEXT 20 ROWS ONLY;
```

---

**Question**: How does Full-Text Search compare to the LIKE operator?

**Answer**: LIKE '%term%' cannot use indexes (table/scan) and has no linguistic awareness. Full-Text Search creates an inverted index, supports word stemming, thesaurus, fuzzy search, and ranking (FREETEXT, CONTAINS). Use Full-Text Search for text-heavy searches and LIKE for simple prefix matches or small tables.

```sql
-- Full-Text Search
SELECT * FROM Documents WHERE CONTAINS(Content, 'FORMSOF(INFLECTIONAL, run)');
-- LIkE (no index use with leading wildcard)
SELECT * FROM Documents WHERE Content LIKE '%running%';
```

---

**Question**: How do you handle JSON and XML in SQL Server?

**Answer**: SQL Server supports OPENJSON and JSON_VALUE to query JSON, and FOR JSON to output results as JSON. For XML, use nodes(), value(), and exist() XQuery methods, with FOR XML PATH for output. JSON is generally simpler and more performant than XML in modern SQL Server.

```sql
-- JSON querying
SELECT JSON_VALUE(@json, '$.customer.name') AS Name;
SELECT * FROM OPENJSON(@json) WITH (Id INT, Name VARCHAR(50));

-- XML querying
SELECT @xml.value('(/root/customer/name)[1]', 'VARCHAR(50)') AS Name;
SELECT * FROM @xml.nodes('/root/customer') AS X(C);
```

---

**Question**: How do you design a database for multi-tenancy?

**Answer**: Three approaches: (1) Separate Database per tenant (best isolation, highest cost, easier backup/restore). (2) Shared Database with Shared Schema and a TenantId discriminator column (lowest cost, requires row-level security). (3) Shared Database with Separate Schema per tenant (schema-per-tenant, moderate isolation). Choose based on compliance, scale, and operational complexity.

```sql
-- Shared schema approach with row-level security
CREATE SECURITY POLICY TenantFilter
ADD FILTER PREDICATE dbo.fn_TenantPredicate(TenantId) ON dbo.Orders;
```

---

**Question**: What are soft deletes vs hard deletes and when would you use each?

**Answer**: Hard DELETE permanently removes rows. Soft deletes set an IsDeleted flag (or DeletedAt timestamp) to mark rows as inactive without removal. Use soft deletes for recoverability, auditing, and referential integrity preservation. Use hard deletes when legal data purging is required or to reclaim storage. Soft deletes require query filters on every read.

```sql
-- Soft delete
UPDATE Orders SET IsDeleted = 1, DeletedAt = GETUTCDATE() WHERE Id = @OrderId;
-- Queries must always filter
SELECT * FROM Orders WHERE IsDeleted = 0;
```

---

**Question**: What are Slowly Changing Dimensions (SCD) Type 1, 2, and 3?

**Answer**: SCD Type 1 overwrites the old value (no history). Type 2 adds a new row with versioning via effective dates or version number (full history). Type 3 stores current and previous value in separate columns (limited history). Type 2 is most common in data warehousing for tracking attribute changes.

```sql
-- SCD Type 2: insert new version on change
UPDATE Customer SET EndDate = GETDATE() WHERE Id = @Id AND EndDate IS NULL;
INSERT INTO Customer (Id, Name, Address, StartDate, Version) 
VALUES (@Id, @NewName, @NewAddress, GETDATE(), @NextVersion);
```

---

**Question**: What is the difference between Star Schema and Snowflake Schema?

**Answer**: Star Schema has a central fact table surrounded by denormalized dimension tables (one level deep, simpler queries, faster reads). Snowflake Schema normalizes dimensions into multiple related tables (less redundancy, more joins, complex queries). Star is preferred for OLAP and reporting; Snowflake is used when dimension hierarchies or storage optimization matter.

---

**Question**: What is the difference between OLTP and OLAP?

**Answer**: OLTP (Online Transaction Processing) handles high-volume, short, concurrent transactions (INSERT/UPDATE/DELETE). OLAP (Online Analytical Processing) handles complex aggregations and historical analysis on large datasets (SELECT-heavy). OLTP databases are normalized, while OLAP databases (data warehouses) are denormalized with star/snowflake schemas.

---

**Question**: What is the difference between ETL and ELT?

**Answer**: ETL (Extract, Transform, Load) transforms data in a staging area before loading into the target. ELT (Extract, Load, Transform) loads raw data first, then transforms in the target (typically a data warehouse). ELT leverages the warehouse's compute power (e.g., Snowflake, BigQuery) for large-scale transformations and is more flexible for agile schema changes.

---

**Question**: What is Change Data Capture (CDC) and how does it work?

**Answer**: CDC captures incremental changes (INSERT, UPDATE, DELETE) from a source table into change tables without affecting the workload. SQL Server CDC reads the transaction log to capture changes asynchronously. Used for data warehousing replication, audit trails, and cache invalidation. Enable per table with sys.sp_cdc_enable_table.

```sql
-- Enable CDC on database
EXEC sys.sp_cdc_enable_db;
-- Enable CDC on a table
EXEC sys.sp_cdc_enable_table @source_schema = 'dbo', @source_name = 'Orders', @role_name = NULL;
-- Query changes
SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_Orders(@from_lsn, @to_lsn, 'all');
```
