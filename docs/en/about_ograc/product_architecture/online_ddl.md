# Online DDL

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:45:19.420Z pushedAt=2026-08-17T00:46:29.616Z -->

## Core Objectives

In conventional database systems, an exclusive lock is placed on a table when its schema is altered (for example, adding a column, creating an index, or modifying a column type). For large tables, such changes may last hours or even days. During this period, all DML and query operations on the database are blocked, rendering the business unavailable. This is unacceptable for services that require 24/7 availability.

Online schema change technology reduces the duration of exclusive-lock phases by separating the long-running build phase from the metadata switch phase. It reduces, but does not eliminate, the impact of schema changes on DML and queries.

## Working Mechanism

### Overall Process

The overall process of altering a table schema follows these steps: Acquire DDL latch (shared lock) -> Execute alteration -> Release DDL latch

For ordinary operations, the target table and its parent and child tables may be locked for the duration of the build and metadata change, so concurrent DML can be blocked. In a multi-write cluster, ordinary object DDL must also complete logical DDL synchronization. `CREATE INDEX ONLINE` and `REBUILD INDEX ONLINE` use a specific online index path and are not supported in cluster mode.

### Shadow Index

Index creation is inherently a resource-intensive task. In addition to significant resource consumption, it can also render the database unavailable for an extended period. During conventional online rebuilds, background index modifications and foreground DML operations interleave, which may increase the probability of deadlocks under extreme conditions.

oGRAC employs the shadow index technique, which separates the "physical creation" and "logical activation" of an index into two distinct phases, with a "buffer" introduced between them. The index is created safely in the background and remains invisible throughout this process. Since the optimizer ignores it, there is no risk of sudden execution plan changes even if the creation process is relatively slow. All DML operations are also applied to the shadow index. Once index creation is complete, the shadow index is brought online through an extremely fast metadata modification operation.

The build scan uses the committed data version visible at the start of the read according to the current SCN, avoiding a long-lived user transaction snapshot. This only defines the scan semantics of index construction; it does not introduce a new transaction isolation level. The shadow index path applies only in non-cluster mode, and its final metadata switch may still wait for concurrent access to finish.

### Parallel DDL

During conventional table alterations, a single process performs all the work, and its drawbacks become particularly pronounced when processing large-scale data. Hardware resources cannot be fully utilized, resulting in low execution efficiency and prolonged blocking, which runs counter to modern operational principles. To address the pain points of traditional table alterations, oGRAC introduces parallel DDL, which leverages multiple threads to execute a single DDL operation, thereby significantly accelerating large-scale DDL operations.

Taking parallel index rebuilding as an example, oGRAC initializes a worker thread pool whose size defaults to the number of CPU cores. Based on the degree of parallelism specified by the `PARALLEL` directive, the coordinator thread dynamically assigns data block ranges of the table or index to worker threads. Each worker thread scans the data within its assigned range, and the coordinator thread merges the results from all worker threads and writes them into the new index segment. The tasks of worker threads are dynamically adjusted at runtime: tasks from overloaded threads are split and processed by idle threads. The entire process fully leverages the parallel computing capabilities of multi-core CPUs.

### Data Dictionary

System tables are metadata tables that manage all objects in the database and serve as the entry point for data access. The data dictionary cache (DC) reads system tables and loads various objects into memory according to a defined schema, facilitating the lookup and management of database objects. The existence of the DC makes system table access transparent and accelerates the access process.

An entity is a concrete representation that holds the actual metadata of a scanned object. Since table schemas are not static, multiple versions of an entity may exist. Each time an object is opened, the latest version of the entity is accessed; therefore, after a DDL operation is executed, the entity is marked as invalid. During execution, the `chg_scn` and `valid` flags are checked, and if the entity is found to be invalid, it is reopened.

## Application Scenarios

| Operation       | Operation | Allow Concurrent DML | Allow Concurrent DQL | Remarks |
|-----------------|--------------------|----------------------|----------------------|---------|
| Secondary index operations | Add secondary index | No | Yes | Ordinary construction holds a table lock and blocks concurrent DML; cluster DDL synchronization is also required. |
|                 | Add secondary index online | Yes | Yes | Non-cluster mode only; the start and final switch phases may still wait. `CREATE INDEX ONLINE` and `REBUILD INDEX ONLINE` are not supported in cluster mode. |
|                 | Delete secondary index | No | Yes | Lock and dictionary refresh phases may cause additional waits for special dependencies. |
|                 | Rename index | No | Yes | Metadata and dictionary refresh phases may briefly wait. |
| Primary key operations | Add primary key | No | Yes | Constraint, index, and dictionary changes are required. |
|                 | Delete primary key | No | Yes | Constraint and dictionary changes are required. |
| Column operations | Add column | No | Yes | A nullable column is usually shorter; defaults or data changes may extend lock holding. |
|                 | Delete column | No | Yes | Dictionary and object-version refresh are required. |
|                 | Rename column | No | Yes | Metadata and dictionary refresh phases may briefly wait. |
|                 | Change column type | No | Yes | May involve data rewrite or object-version switching. |

## Best Practices

### Operate During Off-Peak Hours

Although online table schema changes can reduce concurrency impact, they are not entirely impact-free. During the change, index trees may be built, table data may be read, and new data may be written, resulting in additional CPU or I/O consumption that affects database performance.
