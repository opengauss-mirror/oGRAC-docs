# DROP TABLESPACE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:22.925Z pushedAt=2026-08-17T09:14:36.184Z -->

## Function Description

Deletes a tablespace.

## Precautions

- You must have the DROP TABLESPACE permission to drop a tablespace.

- Tablespaces cannot be dropped in MOUNT mode.

- The system, undo, and temp tablespaces cannot be dropped.

- When you drop a tablespace while retaining the data files, the files are appended with a delete suffix. To free up the space, you must manually delete these files.

## Syntax

**stmt:**

```sql
DROP TABLESPACE name
    [INCLUDING CONTENTS
        [{AND|KEEP} DATAFILES [CASCADE CONSTRAINTS]]
    ]
```

## Parameter Description

- **Without INCLUDING**: Retains data files when dropping a tablespace. If the tablespace to be dropped contains objects such as table indexes or is the default tablespace of a user, the drop operation will raise an error.

- **INCLUDING CONTENTS AND DATAFILES**: Deletes data files when dropping a tablespace.

- **INCLUDING CONTENTS KEEP DATAFILES**: Retains data files when dropping a tablespace.

- **CASCADE CONSTRAINTS**: Deletes related cascades when dropping a tablespace.

## Examples

```
-- 1. Basic drop: remove metadata only, keep physical files
DROP TABLESPACE tbs1;

-- 2. Drop the tablespace including contents and data files
DROP TABLESPACE tbs2 INCLUDING CONTENTS;

-- 3. Drop the tablespace including contents and data files
DROP TABLESPACE tbs3 INCLUDING CONTENTS AND DATAFILES;

-- 4. Drop the tablespace including contents but retain data files
DROP TABLESPACE tbs4 INCLUDING CONTENTS KEEP DATAFILES;

-- 5. Drop the tablespace including all contents, data files, and cascade constraints
DROP TABLESPACE tbs5 INCLUDING CONTENTS AND DATAFILES CASCADE CONSTRAINTS;

-- 6. Drop the tablespace and its contents, retain the data files, and remove cascade constraints
DROP TABLESPACE tbs6 INCLUDING CONTENTS KEEP DATAFILES CASCADE CONSTRAINTS;
```
