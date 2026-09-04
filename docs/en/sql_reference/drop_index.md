# DROP INDEX

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:56.867Z pushedAt=2026-08-17T07:15:28.290Z -->

## Function Description

Deletes an index.

## Precautions

- Dropping an index owned by the current user requires no additional permissions. Dropping an index owned by another user requires the DROP ANY INDEX permission. Indexes in the sys schema can only be dropped by the sys user.

- Dropping indexes is not allowed during database restart recovery.

## Syntax

**stmt:**

```sql
DROP INDEX [IF EXISTS] [schema_name.]index_name
ON
[schema_name.]table_name
```

## Parameter Description

- **[schema_name.]table_name**: name of the table from which the index is to be dropped

## Examples

```
--  Drop an index with the table name specified
DROP INDEX idx_test1_name ON test_table1;

-- Drop an index with schema qualification
DROP INDEX sys.idx_test1_email ON sys.test_table1;
```
