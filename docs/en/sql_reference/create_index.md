# CREATE INDEX

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:36.622Z pushedAt=2026-08-13T09:52:52.273Z -->

## Function Description

Creates an index on a specified table to improve query performance.

## Precautions

- Creating an index requires the CREATE ANY INDEX permission.

- A composite index can contain no more than 16 columns, with a maximum length of 4052.

- Creating a regular index is not supported for the LOB, ARRAY, and IMAGE types. When the parameter of a function-based index is an expression, the result cannot be of the LOB/ARRAY/IMAGE type.

- Function-based indexes support abs, decode, jsonb_value, json_value, lower, nvl, nvl2, radians, regexp_instr, regexp_substr, reverse, substr, substrb, to_char, to_date, to_number, trim, trunc, and upper.

- Partitioned indexes are supported only on partitioned tables. A partitioned table can have both partitioned indexes and global indexes. The number of partitioned indexes must be consistent with the number of partitions.

## Syntax

**stmt:**

```sql
CREATE [UNIQUE] INDEX [IF NOT EXISTS] [schema_name.]index_name ON index_table_clause
    [CRMODE PAGE]
    [PARALLEL n]
    [REVERSE]
    [NOLOGGING]
```

**index_table_clause:**

```
    [schema_name.]table_name
    ({column_name | column_expr }[,...])
    index_attr_clause
```

**index_attr_clause:**

```
    [[TABLESPACE tablespace_name] [index_partition_clause]]
```

**index_partition_clause:**

```
    LOCAL [({PARTITION partition_name [TABLESPACE tablespace_name] [PCTFREE int]}[,...])]
```

## Parameter Description

- **column_expr**: Index expression. Function-based index column expressions support only some function expressions. See [Precautions](#precautions).

- **CRMODE**: MVCC mode. PAGE indicates page-level MVCC. The default is consistent with the CRMODE of the table.

- **PARALLEL**: Degree of parallelism for creating an index. Not supported for function-based indexes, temporary table indexes, or online index creation.

- **REVERSE**: Reverse index.

- **NOLOGGING**: Does not log REDO when creating an index.

- **LOCAL**: Partitioned index, that is, an index is created separately on each partition.

- **PCTFREE**: Percentage of space reserved in an index block for future index entry updates.

## Examples

```
-- Create a regular index on the last_name column of the employees table.
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Create a unique index on the department_name column of the departments table.
CREATE UNIQUE INDEX idx_dept_name ON departments(department_name);

-- Create a composite index on customer and date columns of the orders table.
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);


-- Create a reverse-key index to reduce index block contention.
CREATE INDEX idx_emp_id_reverse ON employees(manager_id) REVERSE;

-- Create a local partitioned index on the partitioned table sales.
CREATE INDEX idx_sales_date_local ON sales(sale_date) LOCAL;

-- Specify the tablespace and attributes for each partition.
CREATE INDEX idx_sales_product_local ON sales(product_id, sale_date) LOCAL
(
    PARTITION p1_2023 TABLESPACE idx_ts1 COMPRESS,
    PARTITION p2_2023 TABLESPACE idx_ts2,
    PARTITION p3_2023 TABLESPACE idx_ts3,
    PARTITION p4_2023 TABLESPACE idx_ts3,
    PARTITION p5_future TABLESPACE idx_ts3
);

-- Create an index on an expression (requires syntax support for idx_column_expr).
CREATE INDEX idx_emp_upper_name ON employees(UPPER(last_name));


-- Parallel index creation for large tables.
CREATE INDEX idx_logs_timestamp ON application_logs(log_timestamp, user_id)
PARALLEL 8
NOLOGGING
TABLESPACE logs_index_ts
CRMODE PAGE;
```
