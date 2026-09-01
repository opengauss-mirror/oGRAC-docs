# CREATE TABLESPACE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:34.910Z pushedAt=2026-08-17T04:51:34.107Z -->

## Function Description

Creates a tablespace.

## Precautions

- The CREATE TABLESPACE permission is required.

- Data files cannot be specified to the log directory.

- It is not recommended for regular users to use the system tablespace.

- The maximum size of a single database file in a tablespace is determined by `PAGE_SIZE`. For UNDO tablespaces, the maximum size = 4 * 1024 * 1024 * `PAGE_SIZE`; for other tablespaces, the maximum size = 1024 * 1024 * 1024 * `PAGE_SIZE`.

## Syntax

**stmt:**

```sql
CREATE [UNDO] TABLESPACE name
    DATAFILE {datafile_clause [, ...]} 
    [NOLOGGING]
    [autooffline_clause]
    [EXTENT AUTOALLOCATE]
```

**datafile_clause:**

```
    file_name SIZE size_clause
    [COMPRESS]
    [autoextend_clause]
```

**autoextend_clause:**

```
    AUTOEXTEND {OFF 
                |   ON [NEXT size_clause] [MAXSIZE {size_clause | UNLIMITED}]
                }
```

**autooffline_clause:**

```
    AUTOOFFLINE [ON|OFF]
```

**size_clause:**

```
    size [K|M|G]
```

- **K**: KB

- **M**: MB

- **G**: GB

## Parameter Description

- **UNDO**: Creates an UNDO tablespace.

- **file_name**: Path to the tablespace data file. An absolute path or a relative path can be specified. A relative path is saved in the data directory. The owner and group of the directory or parent directory where the data file resides must be the same as the database installation user, with permissions set to 700.

- **NOLOGGING**: The tablespace is of the NOLOGGING type.

- **EXTENT AUTOALLOCATE**: Enables automatic extent size allocation for the tablespace.

- **COMPRESS**: Table compression feature. Not applicable in cluster mode.

- **AUTOEXTEND OFF**: Disables automatic extension of the tablespace size. Disabled by default.

- **AUTOEXTEND ON**: Enables automatic extension.

    - **NEXT**: Automatic extension size

    - **MAXSIZE**: Upper limit of automatic extension

    - **UNLIMITED**: Unlimited extension

- **AUTOOFFLINE ON/OFF**: Enables or disables automatic offline of the tablespace, which takes effect at startup. The tablespace is automatically taken offline when a file fails to open. This setting applies only to user-defined tablespaces.

## Examples

```
-- 1. Create a basic tablespace.
CREATE TABLESPACE tbs1
    DATAFILE '/home/ogracdba/data/tbs1.dbf' SIZE 100M;

-- 2. Create a tablespace with automatic extension.
CREATE TABLESPACE tbs2
    DATAFILE '/home/ogracdba/data/tbs2.dbf' SIZE 200M
    AUTOEXTEND ON NEXT 50M MAXSIZE 1G;

-- 3. Create a tablespace with unlimited automatic extension.
CREATE TABLESPACE tbs3
    DATAFILE '/home/ogracdba/data/tbs3.dbf' SIZE 500M
    AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;

-- 4. Create a tablespace with multiple data files.
CREATE TABLESPACE tbs4
    DATAFILE 
        '/home/ogracdba/data/tbs4_1.dbf' SIZE 100M,
        '/home/ogracdba/data/tbs4_2.dbf' SIZE 200M
    EXTENT AUTOALLOCATE;

-- 5. Create an UNDO tablespace.
CREATE UNDO TABLESPACE undotbs
    DATAFILE '/home/ogracdba/data/undotbs.dbf' SIZE 2G
    AUTOEXTEND ON NEXT 500M MAXSIZE 10G;

-- 6. Create a NOLOGGING tablespace.
CREATE TABLESPACE tbs_nolog
    DATAFILE '/home/ogracdba/data/tbs_nolog.dbf' SIZE 300M
    NOLOGGING;

-- 7. Create a compressed tablespace.
CREATE TABLESPACE tbs_compress
    DATAFILE '/home/ogracdba/data/tbs_compress.dbf' SIZE 500M COMPRESS
    AUTOEXTEND ON NEXT 100M;

-- 8. Create a tablespace with auto-offline functionality.
CREATE TABLESPACE tbs_autooffline
    DATAFILE '/home/ogracdba/data/tbs_autooffline.dbf' SIZE 200M
    AUTOOFFLINE ON;

-- 9. Create a tablespace combining multiple features.
CREATE TABLESPACE tbs_complex
    DATAFILE 
        '/home/ogracdba/data/tbs_complex1.dbf' SIZE 300M COMPRESS,
        '/home/ogracdba/data/tbs_complex2.dbf' SIZE 300M
    NOLOGGING
    AUTOOFFLINE ON
    EXTENT AUTOALLOCATE
    AUTOEXTEND ON NEXT 100M MAXSIZE 2G;

-- 10. Create a tablespace using a relative path (relative to the database data directory).
CREATE TABLESPACE tbs_relative
    DATAFILE './tbs_relative.dbf' SIZE 100M;

-- 11. Create a tablespace with automatic extension disabled.
CREATE TABLESPACE tbs_noextend
    DATAFILE '/home/ogracdba/data/tbs_noextend.dbf' SIZE 500M
    AUTOEXTEND OFF;

-- 12. Create a tablespace with automatic offline disabled.
CREATE TABLESPACE tbs_nooffline
    DATAFILE '/home/ogracdba/data/tbs_nooffline.dbf' SIZE 200M
    AUTOOFFLINE OFF;

```
