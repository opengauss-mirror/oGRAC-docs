# CREATE TABLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:07.885Z pushedAt=2026-08-17T04:03:01.790Z -->

## Function Description

Creates a table.

## Precautions

- To create a table for the current user, the CREATE TABLE permission is required. To create a table for another regular user, the CREATE ANY TABLE permission is required. Regular users cannot create SYS tables.

- Auto-increment columns support only the int and bigint types. A table supports only one auto-increment column, and the auto-increment column must be a primary key or a unique index.

- Foreign key references default to referencing the primary key of the parent table. An error is reported if no primary key exists.

- A CHECK constraint can include up to 16 columns.

- To create a local temporary table, `LOCAL_TEMPORARY_TABLE_ENABLED` must be enabled. The table name must start with `#`, and `ON COMMIT DELETE ROWS` is not supported.

- In a temporary table, BLOB is defined as `RAW(8000)`, and CLOB is defined as `VARCHAR(8000B)`.

## Syntax

**stmt:**

```sql
CREATE [[GLOBAL] TEMPORARY] TABLE [IF NOT EXISTS] [schema_name.]table_name
    {({column_def_clause}[,...] [external_constraint][,...])} | {AS query}
```

For the complete definitions of the shared statement tail and each clause, see [CREATE TABLE Common Clauses](shared/create_table_common_clauses.md).

## Parameter Description

- **TEMPORARY**: local temporary table

- **GLOBAL TEMPORARY**: global temporary table

- **ON COMMIT DELETE ROWS**: transaction-level temporary table. Data is cleared when the transaction ends, but the table definition is not deleted. Default behavior

- **ON COMMIT PRESERVE ROWS**: session-level temporary table. Data is cleared when the session ends, but the table definition is not deleted.

- **CRMODE**: MVCC mode. `PAGE` indicates page-level MVCC. The default value is the `CR_MODE` configuration.

- **SERIAL**: auto-increment column. The difference from `AUTO_INCREMENT` is that the default data type of `SERIAL` is BIGINT.

- **DEFAULT expr [ON UPDATE expr]**: default value for a column. `ON UPDATE expr` is a compatibility syntax. When a row is updated and the column is not specified, the update default value is used for filling.

- **COLLATE**: collation. Supported collations include: `UTF8_BIN` (case-sensitive), `UTF8_GENERAL_CI` (case-insensitive), `UTF8_UNICODE_CI` (case-insensitive), `GBK_BIN` (case-sensitive), and `GBK_CHINESE_CI` (case-insensitive)

- **REFERENCES [schema_name.]table_name[(column_name)] ON DELETE CASCADE**: foreign key cascade setting. When a row in the referenced table is deleted, the corresponding rows in this table are also deleted.

- **REFERENCES [schema_name.]table_name[(column_name)] ON DELETE SET NULL**: foreign key cascade setting. When a row in the referenced table is deleted, the corresponding rows in this table are set to NULL.

- **USING INDEX**: index attributes for a constraint

- **INITRANS**: the initial number of transaction slots in the database

- **MAXTRANS**: the maximum number of transaction slots in the database

- **FORMAT ({ASF|CSF})**: row format. The default is Aligned Stream Format (ASF). Temporary tables do not support Compact Stream Format (CSF).

- **STORAGE ({INITIAL int [K|M|G|T] | MAXSIZE {UNLIMITED | int [K|M|G|T]}}[ ...])**: `INITIAL` specifies the initial size of the table, `MAXSIZE` specifies the maximum storage size of the table, and `UNLIMITED` indicates unlimited storage.

- **RECORDS DELIMITED BY records_delimiter FIELDS TERMINATED BY fields_term**: Specifies the record delimiter and field delimiter for external tables. `records_delimiter` supports a single character or newline, and `fields_term` supports a single character.

- **ORGANIZATION EXTERNAL**: external table-related settings. External table columns do not support the LOB type.

- **TYPE LOADER**: database conversion type. `LOADER` indicates text conversion.

- **DIRECTORY**: The name of the directory where the external table resides, which must be created in advance using CREATE DIRECTORY.

- **ACCESS PARAMETERS**: conversion parameters

- **LOCATION**: file name

- **AUTO_INCREMENT [=] value**: Auto-increment initial value

- **APPENDONLY {ON|OFF}**: Append-only write. When enabled, concurrent writes to the same table from different threads allocate new pages for each write, reducing lock contention. However, this may result in significant page space waste. The default value is `OFF`.

- **LOB (LOB_item) STORE AS**: Specifies that the LOB column (`LOB_item`) is stored in a separate segment.

    - **ENABLE STORAGE IN ROW**: In-row storage

    - **DISABLE STORAGE IN ROW**: Out-of-row storage

## Examples

```
-- 1. Simple employee table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10,2) CHECK (salary > 0),
    dept_id INT
);

-- 2. With foreign key and comments
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) COMMENT 'Department Name',
    manager_id INT
);

-- Global temporary table
CREATE GLOBAL TEMPORARY TABLE temp_data (
    session_id VARCHAR(50)
) ON COMMIT DELETE ROWS;

-- Session-level temporary table
CREATE TEMPORARY TABLE #session_cache (
    key VARCHAR(100) PRIMARY KEY,
    value TEXT
) ON COMMIT PRESERVE ROWS;

-- Column-level constraint
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 0),
    status VARCHAR(10) DEFAULT 'ACTIVE'
);

-- Table-level foreign key
CREATE TABLE orders (
    order_id SERIAL,
    user_id INT,
    CONSTRAINT pk_order PRIMARY KEY (order_id),
    CONSTRAINT fk_user FOREIGN KEY (user_id) 
        REFERENCES users(user_id) ON DELETE CASCADE
);

-- Specify tablespace and storage
CREATE TABLE large_logs (
    log_id SERIAL PRIMARY KEY,
    log_time TIMESTAMP,
    message TEXT
) TABLESPACE log_ts
  PCTFREE 10
  STORAGE (INITIAL 100M MAXSIZE 2G);

-- Table with BLOB
CREATE TABLE documents (
    doc_id SERIAL PRIMARY KEY,
    doc_content BLOB
) LOB (doc_content) STORE AS (TABLESPACE lob_ts);

-- CATS
CREATE TABLE sales_summary
AS
SELECT product_id, SUM(quantity) as total_qty
FROM sales
GROUP BY product_id;

-- Create a page-level MVCC table
CREATE TABLE page_mvcc_table (
    id SERIAL PRIMARY KEY,
    log_data TEXT,
    created_time TIMESTAMP
) CRMODE PAGE;

-- Specify only physical attributes and use the default tablespace
CREATE TABLE logs (
    log_id SERIAL PRIMARY KEY,
    log_time TIMESTAMP
)
PCTFREE 5
INITRANS 2
MAXTRANS 255
STORAGE (INITIAL 100M MAXSIZE 1G);

-- Create a table in ASF format
CREATE TABLE user_profiles (
    user_id SERIAL PRIMARY KEY,
    profile_data JSONB,
    preferences JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
FORMAT ASF;

-- Specify only DIRECTORY and LOCATION
CREATE TABLE external_simple (
    id INT,
    name VARCHAR(50)
)
ORGANIZATION EXTERNAL (
    DIRECTORY data_dir
    LOCATION 'simple.csv'
);
```
