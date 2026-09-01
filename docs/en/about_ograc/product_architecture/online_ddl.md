# Online DDL

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:45:19.420Z pushedAt=2026-08-17T00:46:29.616Z -->

## Core Objectives

In conventional database systems, an exclusive lock is placed on a table when its schema is altered (for example, adding a column, creating an index, or modifying a column type). For large tables, such changes may last hours or even days. During this period, all DML and query operations on the database are blocked, rendering the business unavailable. This is unacceptable for services that require 24/7 availability.

Online schema change technology enables modifications to table definitions with no or minimal impact on the normal service of the database. It is key to ensuring smooth database evolution and business continuity.

## Working Mechanism

### Overall Process

The overall process of altering a table schema follows these steps: Acquire DDL latch (shared lock) -> Execute alteration -> Release DDL latch

Typically, before an alteration operation begins, a table-level exclusive lock is added to the target table and its parent and child tables, and the lock is released after the alteration is completed. All DML operations are blocked while the table lock is held. oGRAC has implemented extensive optimizations to address this issue.

### Shadow Index

Index creation is inherently a resource-intensive task. In addition to significant resource consumption, it can also render the database unavailable for an extended period. During conventional online rebuilds, background index modifications and foreground DML operations interleave, which may increase the probability of deadlocks under extreme conditions.

oGRAC employs the shadow index technique, which separates the "physical creation" and "logical activation" of an index into two distinct phases, with a "buffer" introduced between them. The index is created safely in the background and remains invisible throughout this process. Since the optimizer ignores it, there is no risk of sudden execution plan changes even if the creation process is relatively slow. All DML operations are also applied to the shadow index. Once index creation is complete, the shadow index is brought online through an extremely fast metadata modification operation.

Background index creation does not require a table-level exclusive lock, and the metadata modification operation takes only milliseconds. Through the shadow index technique, oGRAC implements an online index creation method that completely avoids blocking DML operations, representing a more advanced and safer implementation pattern of online DDL.

### Parallel DDL

During conventional table alterations, a single process performs all the work, and its drawbacks become particularly pronounced when processing large-scale data. Hardware resources cannot be fully utilized, resulting in low execution efficiency and prolonged blocking, which runs counter to modern operational principles. To address the pain points of traditional table alterations, oGRAC introduces parallel DDL, which leverages multiple threads to execute a single DDL operation, thereby significantly accelerating large-scale DDL operations.

Taking parallel index rebuilding as an example, oGRAC initializes a worker thread pool whose size defaults to the number of CPU cores. Based on the degree of parallelism specified by the `PARALLEL` directive, the coordinator thread dynamically assigns data block ranges of the table or index to worker threads. Each worker thread scans the data within its assigned range, and the coordinator thread merges the results from all worker threads and writes them into the new index segment. The tasks of worker threads are dynamically adjusted at runtime: tasks from overloaded threads are split and processed by idle threads. The entire process fully leverages the parallel computing capabilities of multi-core CPUs.

### Data Dictionary

System tables are metadata tables that manage all objects in the database and serve as the entry point for data access. The data dictionary cache (DC) reads system tables and loads various objects into memory according to a defined schema, facilitating the lookup and management of database objects. The existence of the DC makes system table access transparent and accelerates the access process.

An entity is a concrete representation that holds the actual metadata of a scanned object. Since table schemas are not static, multiple versions of an entity may exist. Each time an object is opened, the latest version of the entity is accessed; therefore, after a DDL operation is executed, the entity is marked as invalid. During execution, the `chg_scn` and `valid` flags are checked, and if the entity is found to be invalid, it is reopened.

## Application Scenarios

| Operation       | Operation | Allow Concurrent DML | Allow Concurrent DQL | Remarks |
|-----------------|--------------------|----------------------|----------------------|---------|
| Secondary index operations | Add secondary index | Yes | Yes | Transient impact on DML at the start and end phases |
|                 | Delete secondary index | No | Yes |  |
|                 | Rename index | No | Yes | Only metadata modification, transient impact on DML |
| Primary key operations | Add primary key | No | Yes |  |
|                 | Delete primary key | No | Yes |  |
| Column operations | Add column | No | Yes | If adding a NULL column, transient impact on DML |
|                 | Delete column | No | Yes |  |
|                 | Rename column | No | Yes | Only metadata modification, transient impact on DML |
|                 | Change column type | No | Yes |  |

## Best Practices

### Operate During Off-Peak Hours

Although online table schema changes can reduce concurrency impact, they are not entirely impact-free. During the change, index trees may be built, table data may be read, and new data may be written, resulting in additional CPU or I/O consumption that affects database performance.
