# CREATE TABLE PARTITION

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:06.202Z pushedAt=2026-08-17T03:21:30.138Z -->

## Function Description

Creates a partitioned table.
Partitioning is a technique that divides a logically large table into multiple smaller, more manageable parts (called "partitions" or "subtables") in physical storage. Each partition can be independently stored, backed up, maintained, and queried, yet it still appears as a complete table to users and apps. oGRAC supports range partitioning, list partitioning, hash partitioning, and interval partitioning.

## Precautions

Some attributes of partitioned tables are incompatible with those of regular tables.

- Currently, four partitioning types are supported: RANGE, LIST, HASH, and INTERVAL.

- A partition key can contain no more than 16 columns.

- Column types supported as partition keys: UINT32, UINT64, INTEGER, BIGINT, REAL, NUMBER, NUMBER2, NUMBER3, DECIMAL, DATE, TIMESTAMP, INTERVAL_DS, INTERVAL_YM, CHAR, VARCHAR, STRING, BINARY, RAW

- A maximum of 16,777,216 partitions is supported, and a maximum of 8,388,608 partitions is supported when HASH partitioning is used.

## Syntax

**stmt:**

```sql
CREATE TABLE [IF NOT EXISTS] [schema_name.]table_name
    ({column_def_clause}[,...] [external_constraint][,...])
```

For the complete definitions of the shared statement tail and each clause, see [CREATE TABLE Common Clauses](shared/create_table_common_clauses.md).

A partitioned table inherits the column definitions, constraints, and storage clauses of a regular table.

**using_index_clause:**

```
    USING INDEX [ INITRANS int
                | TABLESPACE tablespace_name
                | LOCAL [({PARTITION partition_name [TABLESPACE tablespace_name | INITRANS int | PCTFREE int | ({SUBPARTITION subpartition_name[TABLESPACE tablespace_name]} [,...] )]}[,...])]
              ] [ ...]
```

**table_attr_clause:**

```
    [column_attr_clause]
    [AUTO_INCREMENT [=] value]
    [table_partition_clause]
```

**column_attr_clause:**

```
    [LOB (LOB_item) STORE AS LOB_segname [(LOB_parameters)]]
    [APPENDONLY {ON|OFF}]
```

**table_partition_clause:**

```
    range_partition_clause
    | list_partition_clause
    | hash_partition_clause
    | interval_partition_clause 
```

**range_partition_clause:**

```
    PARTITION BY RANGE (partition_key[,...]) 
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    (range_partition_item[,...])

```

**range_partition_item:**

```
    PARTITION partition_name VALUES LESS THAN ({value | MAXVALUE}[,...]) [physical_properties_clause] [(subpartition_item[,...])]
```

**subpartition_item:**

```
    SUBPARTITION subpartition_name [{VALUES LESS THAN ({value | MAXVALUE}[,...]) | VALUES ({value | DEFAULT}[,...])}] [TABLESPACE tablespace_name]
```

**list_partition_clause:**

```
    PARTITION BY LIST (partition_key[,...])
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    (list_partition_item[,...])
```

**list_partition_item:**

```
    PARTITION partition_name VALUES ([value][,...] [DEFAULT]) [physical_properties_clause] [(subpartition_item[,...])]
```

**hash_partition_clause:**

```
    PARTITION BY HASH (partition_key[,...])
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    ({hash_partition_item1[,...] | hash_partition_item2})
```

**hash_partition_item1:**

```
    PARTITION partition_name  [physical_properties_clause] [(subpartition_item[,...])]
```

**hash_partition_item2:**

```
    PARTITIONS partition_count (STORE IN (tablespace_name[,...]))
```

**interval_partition_clause:**

```
    PARTITION BY RANGE (partition_key) INTERVAL (value)
    [STORE IN (TABLESPACE tablespace_name[,...])]
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    (range_partition_item[,...])
```

**physical_properties_clause:**

```
    segment_attr_clause
    | FORMAT row_format_clause
```

## Parameter Description

- [Parameters common to regular tables](create_table.md#parameter-description)

- `range_partition_clause`: RANGE partitioning

- `list_partition_clause`: LIST partitioning

- `hash_partition_clause`: HASH partitioning

- `interval_partition_clause`: INTERVAL partition

- `partition_key`: the set of columns that constitute the partition key

- `VALUES LESS THAN`: the maximum value of the partition key for a RANGE partition

- `SUBPARTITION BY {RANGE|LIST|HASH}`: the sub-partitioning method

- `MAXVALUE`: a special partition definition representing the maximum value

- `VALUES(value)`: a value for a LIST partition key

- `VALUES(DEFAULT)`: DEFAULT partition where default values are stored

- `PARTITIONS partition_count`: Number of HASH partitions. After specification, `partition_count` partitions are automatically created, with data evenly distributed

## Examples

```
-- Range partitioning

CREATE TABLE sales_range (
sale_id NUMBER,
sale_date DATE,
amount NUMBER,
region VARCHAR2(50)
)
PARTITION BY RANGE (sale_date) (
PARTITION sales_q1 VALUES LESS THAN (TO_DATE('2024-04-01', 'YYYY-MM-DD')),
PARTITION sales_q2 VALUES LESS THAN (TO_DATE('2024-07-01', 'YYYY-MM-DD')),
PARTITION sales_q3 VALUES LESS THAN (TO_DATE('2024-10-01', 'YYYY-MM-DD')),
PARTITION sales_q4 VALUES LESS THAN (MAXVALUE)
);

-- List partitioning

CREATE TABLE employees_list (
emp_id NUMBER,
emp_name VARCHAR2(100),
department VARCHAR2(50),
salary NUMBER
)
PARTITION BY LIST (department) (
PARTITION dept_sales VALUES ('SALES', 'MARKETING'),
PARTITION dept_tech VALUES ('IT', 'ENGINEERING'),
PARTITION dept_hr VALUES ('HR', 'ADMIN'),
PARTITION dept_other VALUES (DEFAULT)
);

-- Hash partitioning method 1: Specify partition names

CREATE TABLE products_hash (
product_id NUMBER,
product_name VARCHAR2(200),
category VARCHAR2(100)
)
PARTITION BY HASH (product_id) (
PARTITION p1,
PARTITION p2,
PARTITION p3,
PARTITION p4
);

-- Hash partitioning method 2: Specify the number of partitions

CREATE TABLE orders_hash (
order_id NUMBER,
order_date DATE,
customer_id NUMBER
)
PARTITION BY HASH (order_id)
PARTITIONS 8;

-- Interval partitioning
CREATE TABLE sales_interval (
sale_id NUMBER,
sale_date DATE,
amount NUMBER,
region VARCHAR2(50)
)
PARTITION BY RANGE (sale_date)
INTERVAL (NUMTOYMINTERVAL(1, 'MONTH'))
(
PARTITION sales_historical VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
PARTITION sales_jan_2024 VALUES LESS THAN (TO_DATE('2024-02-01', 'YYYY-MM-DD'))
);

-- Sub-partitioning RANGE + LIST

CREATE TABLE sales_subpart (
    sale_id      NUMBER,
    sale_date    DATE,
    region       VARCHAR2(20),
    product_type VARCHAR2(30),
    amount       NUMBER
)
PARTITION BY RANGE (sale_date)
SUBPARTITION BY LIST (region)
(
    PARTITION sales_2023 VALUES LESS THAN (DATE '2024-01-01')
    (
        SUBPARTITION north VALUES ('NORTH'),
        SUBPARTITION south VALUES ('SOUTH'),
        SUBPARTITION east VALUES ('EAST'),
        SUBPARTITION west VALUES ('WEST'),
        SUBPARTITION other VALUES (DEFAULT)
    ),
    PARTITION sales_2024 VALUES LESS THAN (DATE '2025-01-01'),
    PARTITION sales_future VALUES LESS THAN (MAXVALUE)
);
```