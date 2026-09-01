# TRUNCATE TABLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:35:45.787Z pushedAt=2026-07-29T06:14:15.823Z -->

## Function Description

Clears table data and releases storage.

## Precautions

- Truncating tables of other users requires the `TRUNCATE ANY TABLE` privilege.

- The `TRUNCATE` statement cannot be rolled back.

- `TRUNCATE` is not supported during database restart rollback.

## Syntax

**stmt:**

```sql
TRUNCATE TABLE [schema_name.]table_name [PURGE] [{DROP|REUSE} STORAGE]
```

## Parameter Description

- **Without specifying [PURGE] [{DROP|REUSE} STORAGE]**: The table is truncated and placed in the recycle bin, and data can be recovered via flashback.

- **PURGE**: Truncates the table and reclaims space.

- **DROP STORAGE**: Truncates the table and reclaims space.

- **REUSE STORAGE**: Truncates the table without reclaiming space; the space is retained for the table's own use.

- **Basic TRUNCATE**: Space is reclaimed when the recycle bin is purged, and data can be recovered via flashback. Suitable for testing and development environments.

- **TRUNCATE PURGE**: Immediately reclaims space, and data is unrecoverable. Suitable for scenarios where storage is tight and data recovery is not required.

- **TRUNCATE DROP STORAGE**: Immediately reclaims space, and data is unrecoverable. Suitable for scenarios where storage is tight and data recovery is not required.

- **TRUNCATE REUSE STORAGE**: Does not reclaim space, and data is unrecoverable. Suitable for tables that are frequently truncated and reloaded.

## Examples

```
-- Truncate the table.
TRUNCATE TABLE test_data;

-- Use PURGE to completely truncate (without entering the recycle bin).
TRUNCATE TABLE sensitive_data PURGE;

-- Use DROP STORAGE to truncate and release space.
TRUNCATE TABLE large_table DROP STORAGE;

-- Use REUSE STORAGE to quickly truncate (retain allocated space).
TRUNCATE TABLE cache_table REUSE STORAGE;

```
