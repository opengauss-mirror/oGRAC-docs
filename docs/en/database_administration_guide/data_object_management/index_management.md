# Index Management

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:00:29.353Z pushedAt=2026-08-24T07:40:51.722Z -->

An index is a database object that establishes an ordered data structure for one or more columns in a table, enabling the database engine to quickly locate data rows that satisfy a condition without scanning the entire table. Creating appropriate indexes is a core technique for improving data retrieval speed in a database.

## Index Creation

By the number of columns an index occupies, indexes can be divided into single-column indexes and composite indexes. By index type, they can be divided into regular indexes, function-based indexes, unique indexes (including primary keys), and partitioned indexes. `oGRAC` supports the above index types, and the creation methods for each type are as follows.

### Basic Syntax

```sql
CREATE INDEX INDEX_NAME ON [Schema.]TABLE_NAME ({COLUMN_NAME | (COLUMN_NAME1, COLUMN_NAME2, ...) | EXPRESSION | FUNCTION}) [TABLESPACE TABLESPACE_NAME];
```

- `INDEX_NAME`: the name of the index, user-defined, and must be unique within a table.

- `Schema`: an optional parameter that specifies the schema to which the index belongs. If not specified, the schema of the current user is used by default.

- `TABLE_NAME`: the name of the table to which the index belongs.

- `COLUMN_NAME`: the name of the column on which the index is based. When creating a composite index, multiple column names are separated by commas.

- `EXPRESSION`: An index based on an expression.

- `FUNCTION`: An index based on a function.

- `TABLESPACE OGRACSPACE`: An optional parameter that specifies the tablespace where the index resides. The current tablespace is used by default.

### Example

```sql
DROP TABLE IF EXISTS USERS;
CREATE TABLE USERS 
(
    ID INT CONSTRAINT PK_IDX PRIMARY KEY, -- A primary key can be specified directly when creating a table, and it is more like a constraint.
    AGE INT ,
    NAME VARCHAR(10),
    ADDRESS VARCHAR(50),
    SALARY NUMBER(20,4),
    USER_ID INT
) TABLESPACE OGRACSPACE;

-- Users can create an index using CREATE INDEX.
CREATE INDEX AGE_IDX ON USERS(AGE) TABLESPACE OGRACSPACE; -- Single-column regular index
CREATE INDEX ADDRESS_UPPER_IDX ON USERS(UPPER(ADDRESS)) TABLESPACE OGRACSPACE; -- Function-based index
CREATE INDEX ID_AGE_IDX2 ON USERS(ID, AGE) TABLESPACE OGRACSPACE; -- Composite index
CREATE UNIQUE INDEX USER_IDX ON USERS(USER_ID) TABLESPACE OGRACSPACE; -- Unique index
```

Note: `oGRAC` requires that the maximum number of columns in a composite index is `16`.

## Partitioned Index Creation

A partitioned index is an index created on a partitioned table, and its creation method is the same as that of an index on a regular table.

```sql
CREATE TABLE orders (
    order_id    NUMBER,
    order_date  DATE,
    customer_id NUMBER,
    region      VARCHAR2(10),
    amount      NUMBER
)
PARTITION BY RANGE (order_date)
SUBPARTITION BY LIST (region)
(
  PARTITION p_2024_q1 VALUES LESS THAN (DATE '2024-04-01')
    (SUBPARTITION p_2024_q1_east VALUES ('EAST'),
     SUBPARTITION p_2024_q1_west VALUES ('WEST')),
  PARTITION p_2024_q2 VALUES LESS THAN (DATE '2024-07-01')
    (SUBPARTITION p_2024_q2_east VALUES ('EAST'),
     SUBPARTITION p_2024_q2_west VALUES ('WEST'))
);

-- Create a regular index on a partitioned table.
CREATE INDEX idx_order_date ON orders(order_date);
-- Create a composite index on a partitioned table.
CREATE INDEX idx_order_date_region ON orders(order_date, region);
-- Create a unique index on a partitioned table.
CREATE UNIQUE INDEX idx_order_id ON orders(order_id);
-- Create a function-based index on a partitioned table.
CREATE INDEX idx_region_upper ON orders(UPPER(region));
```

## Index Rebuild

When a table undergoes frequent `INSERT`, `UPDATE`, and `DELETE` operations, the index is dynamically adjusted accordingly, resulting in space fragmentation in the physical files on disk. The logical order of index entries becomes inconsistent with their physical storage order, which affects sequential scan performance. Index rebuild refers to the process of deleting the old index structure and recreating a new, clean index according to the original definition. This process reorganizes the data pages of the index, eliminates fragmentation, and optimizes the storage structure. Users can rebuild an index using the following command:

```sql
ALTER INDEX INDEX_NAME ON TABLE_NAME REBUILD;
```

Index rebuild is not supported during database restart recovery.

## Index Modification

`oGRAC` allows renaming an existing index, but does not support altering its structure.

```sql
ALTER INDEX AGE_IDX ON USERS RENAME TO AGE_IDX_NEW;

ALTER INDEX idx_order_date ON orders RENAME TO idx_order_date_new;
```

## Index Deletion

Users can delete an index using `DROP INDEX`. Index deletion is not supported during database restart recovery. When a table is deleted, its indexes are deleted as well.

```sql
-- Delete an index on a regular table.
DROP INDEX AGE_IDX ON USERS;
DROP INDEX USER_IDX ON USERS;

-- Delete an index on a partitioned table.
DROP INDEX idx_order_date ON orders;
DROP INDEX idx_order_date_region ON orders;

-- To delete a primary key, delete its associated constraint.
ALTER TABLE USERS DROP CONSTRAINT PK_IDX;
```

In some scenarios, the primary key constraint does not need to be explicitly named. For example:

```sql
DROP TABLE IF EXISTS TEST;
CREATE TABLE TEST(COL1 INT PRIMARY KEY, COL2 INT);
```

In this case, you can view the name of the primary key through `ADM_INDEXES` or `DB_INDEXES` and then delete it. An example is as follows:

```sql
SQL> SELECT INDEX_NAME FROM ADM_INDEXES WHERE TABLE_NAME = 'TEST' AND IS_PRIMARY = 'Y';

INDEX_NAME                                                      
----------------------------------------------------------------
_PK_SYS_1_27                                                    

1 rows fetched.

SQL> ALTER TABLE TEST DROP CONSTRAINT _PK_SYS_1_27;

Succeed.
```

## Index Invalidation

Users can invalidate an index by using the `UNUSABLE` command. After an index is invalidated, the database will not maintain the index structure while performing operations such as insert, delete, and update on the table. If users want to re-enable the index later, they must rebuild the index through a command (the index takes effect automatically after the index rebuild) or delete the index and re-create it.

```sql
ALTER INDEX AGE_IDX ON USERS UNUSABLE; -- Invalidate the index.
ALTER INDEX AGE_IDX ON USERS REBUILD;  -- Rebuild the index.

ALTER INDEX idx_order_date ON orders UNUSABLE; -- Invalidate the index on a partitioned table.
ALTER INDEX idx_order_date ON orders REBUILD;  -- Rebuild the index on the partitioned table.
```

## Index Information View

Users can view index-related information through `ADM_INDEXES` or `DB_INDEXES`.

```sql
SQL> SELECT INDEX_NAME, INDEX_TYPE, TABLESPACE_NAME, IS_PRIMARY, IS_UNIQUE, COLUMNS FROM ADM_INDEXES WHERE TABLE_NAME = 'USERS';

INDEX_NAME                                                       INDEX_TYPE TABLESPACE_NAME                                                  IS_PRIMARY IS_UNIQUE COLUMNS                                                         
---------------------------------------------------------------- ---------- ---------------------------------------------------------------- ---------- --------- ----------------------------------------------------------------
PK_IDX                                                           NORMAL     OGRACSPACE                                                       Y          N         ID                                                              
USER_IDX                                                         NORMAL     OGRACSPACE                                                       N          Y         USER_ID                                                         
ID_AGE_IDX2                                                      NORMAL     OGRACSPACE                                                       N          N         ID, AGE                                                         
ADDRESS_UPPER_IDX                                                NORMAL     OGRACSPACE                                                       N          N         UPPER(ADDRESS)                                                  
AGE_IDX                                                          NORMAL     OGRACSPACE                                                       N          N         AGE                                                             

5 rows fetched.

SQL> SELECT INDEX_NAME, INDEX_TYPE, TABLESPACE_NAME, IS_PRIMARY, IS_UNIQUE, COLUMNS FROM ADM_INDEXES WHERE TABLE_NAME = 'ORDERS';

INDEX_NAME                                                       INDEX_TYPE TABLESPACE_NAME                                                  IS_PRIMARY IS_UNIQUE COLUMNS                                                         
---------------------------------------------------------------- ---------- ---------------------------------------------------------------- ---------- --------- ----------------------------------------------------------------
IDX_ORDER_ID                                                     NORMAL     SYSTEM                                                           N          Y         ORDER_ID                                                        
IDX_ORDER_DATE_REGION                                            NORMAL     SYSTEM                                                           N          N         ORDER_DATE, REGION                                              
IDX_ORDER_DATE_NEW                                               NORMAL     SYSTEM                                                           N          N         ORDER_DATE                                                      
IDX_REGION_UPPER                                                 NORMAL     SYSTEM                                                           N          N         UPPER(region)                                                   

4 rows fetched.

```

- `INDEX_NAME`: index name

- `INDEX_TYPE`: index type

- `TABLESPACE_NAME`: name of the tablespace where the index resides

- `IS_PRIMARY`: Whether it is a primary key index. `Y` indicates yes, and `N` indicates no.

- `IS_UNIQUE`: Whether it is a unique index. `Y` indicates yes, and `N` indicates no.

- `COLUMNS`: names of the columns contained in the index, separated by commas.

## Index Scan Methods

The index scan methods currently supported by `oGRAC` are as follows:

- Index Unique Scan: An equality query that uses all columns of a unique index in the condition.

- Index Range Scan: Used for non‑unique indexes, or unique indexes where not all columns are provided. The condition includes range or equality operators such as =, >, >=, <, <=, BETWEEN, and IN.

- Index Full Scan: Typically scans the index in the physical storage order, independent of query conditions.

- Index Fast Full Scan: Uses block‑level scanning and requires that all queried columns are included in the index. The scan result is unordered. This conflicts with `ORDER BY` when index ordering is expected, and also conflicts with the `hint_no_ffs` hint.

- Index Skip Scan: Used when the leading column(s) of an index are not provided in the query condition — typically when a range predicate appears before an equality predicate.
