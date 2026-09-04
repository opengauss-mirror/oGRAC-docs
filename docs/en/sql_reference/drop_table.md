# DROP TABLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:10.684Z pushedAt=2026-08-17T08:47:21.589Z -->

## Function Description

Deletes a table.

## Precautions

- To drop a table owned by another user, the DROP ANY TABLE permission is required. Tables owned by the SYS user can only be dropped by the SYS user.

- After the recycle bin function is enabled, dropping a table does not immediately delete it. Data can be restored using the flashback command. Tables in the system, nologging, sysaux, and temporary tablespaces do not support flashback.

- Dropping tables is not supported during oGRAC restart recovery.

## Syntax

**stmt:**

```sql
DROP [TEMPORARY] TABLE
[IF EXISTS] [schema_name.]table_name
[CASCADE CONSTRAINTS]
[PURGE]
```

## Parameter Description

- **TEMPORARY**: Specifies that the table is a temporary table.

- **CASCADE CONSTRAINTS**: Deletes foreign key references when the table is dropped.

- **PURGE**: Drops the table without sending it to the recycle bin.

## Examples

```
-- Drop a single table
DROP TABLE employees;

-- Drop a table in a specified schema
DROP TABLE sys.session_cache;

-- Drop a temporary table
DROP TEMPORARY TABLE #session_cache;

-- Drop a table and cascade to foreign key constraints
DROP TABLE departments CASCADE CONSTRAINTS;

-- Permanently drop a table without sending it to the recycle bin
DROP TABLE temporary_data PURGE;

```
