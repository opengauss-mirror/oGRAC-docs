# FLASHBACK TABLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:37:26.028Z pushedAt=2026-07-29T06:14:15.865Z -->

## Feature Description

The `FLASHBACK` feature allows users to quickly recover, without the need for backups, accidentally deleted or modified data.

## Precautions

- Execution requires the `FLASHBACK` privilege. To flash back tables owned by other users, the `FLASHBACK ANY TABLE` privilege is required.

- `FLASHBACK TABLE` supports flashback from the recycle bin and flashback from `UNDO` logs. Tables that have been truncated/dropped are flashed back from the recycle bin, while newly added or updated object data is flashed back from `UNDO`.

## Syntax

```sql
FLASHBACK TABLE [schema_name.]table_name
TO { 
    SCN scn_expr 
    | TIMESTAMP ts_expr 
    | BEFORE {DROP [RENAME TO table_name] | TRUNCATE FORCE}
}
```

## Parameter Description

- `[schema_name.]table_name`: specifies the table to be flashed back. Materialized views, external tables, system tables, standalone partitions, and subpartition tables cannot be flashed back. Tables whose structure has been altered by DDL operations cannot be flashed back from `UNDO`.

- `SCN scn_expr`: flashes back the table from `UNDO` to the point in time specified by the system change number (SCN) expression `scn_expr`.

- `TIMESTAMP ts_expr`: flashes back the table from `UNDO` to the point in time specified by the timestamp expression `ts_expr`.

- `BEFORE DROP`: recovers a dropped table.

- `BEFORE DROP RENAME TO`: recovers a dropped table and specifies a new name for it.

- `BEFORE TRUNCATE FORCE`: recovers a table to its state before truncation. Tables truncated with `PURGE`, `DROP STORAGE`, or `REUSE STORAGE` option cannot be flashed back.

## Examples

```

-- -- Restore a dropped table.
flashback table tb1 to before drop;

-- -- Restore a dropped table and rename it.
flashback table tb1 to before drop rename to new_tb1;

-- -- Restore a truncated table.
flashback table tb1 to before truncate force;

-- -- Query the recycle bin.
SELECT * FROM SYS_RECYCLEBIN;

-- Flashback by SCN.
flashback table tb1 to scn 138953777287169;

-- Flashback from a specified time.
flashback table tb1 to timestamp to_date('2025-12-23 21:11:29');

```

## Related Views

### SYS_RECYCLEBIN

- ID: Object ID

- NAME: Object name

- USER#: User ID

- ORG_NAME: Original object name

- PARTITION_NAME: Name of the object partition

- TYPE#: Object type. 0-5 represent table, index, LOB, partitioned table, partitioned index, and partitioned LOB, respectively.

- OPERATION#: Operation type

- SPACE#: Tablespace ID

- ENTRY: Entry page ID

- FLAGS: `0x0001`, indicating whether the object is valid for flashback, `0x0010`, indicating whether the object is marked for purge, `0x0100`, indicating whether the object has constraints, and `0x1000`, indicating whether the object is encoded.

- ORG_SCN: SCN at object creation

- REC_SCN: SCN at object recycle

- TCHG_SCN: SCN at object definition change

- BASE_ID: Base object ID

- PURGE_ID: Object ID
