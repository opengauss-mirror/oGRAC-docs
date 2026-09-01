# Tablespace Management

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:01:00.336Z pushedAt=2026-08-20T08:31:57.557Z -->

A tablespace (`TABLESPACE`) is a logical container used in a database to manage physical storage space. It serves as the bridge between database objects and disk files. A tablespace in `oGRAC` consists of one or more data files, and a data file belongs exclusively to one tablespace. From the perspective of database storage logic, all data objects in the database reside in tablespaces; from the perspective of physical storage, they are all stored in the corresponding data files. A database can contain multiple tablespaces, and tablespaces support basic functions such as creation, deletion, and capacity extension. A tablespace also belongs exclusively to one database.

## Tablespace Types

- `SYSTEM` tablespace: Stores database metadata. It does not support automatic extension. Storing user data in the `SYSTEM` tablespace is not recommended. If the `SYSTEM` tablespace is full, you need to extend the tablespace to meet requirements.

- `TEMP` tablespace: Automatically maintained by the database. It is mainly used to allocate temporary segments for swapping in and out when executing `SQL` statements that generate a large amount of temporary data, such as creating indexes, sort operations that cannot be completed in memory, and intermediate result sets of `SQL` statements.

- `UNDO` tablespace: Stores `Undo` data. When executing `DML` operations such as insert, update, and delete, the old data before execution is written to the `UNDO` tablespace. It is mainly used for `MVCC` visibility determination, transaction rollback, consistent read, and other operations.

- `USERS` tablespace: The default user tablespace. If no tablespace is explicitly specified when creating a new user, all information of the new user will be stored in the `USERS` tablespace.

- `SYSAUX` tablespace: automatically created when the database is created, mainly storing data related to `WSR` tables and `CBO` statistics tables.

- `TEMP2`: stores data of `NOLOGGING` tables.

- `TEMP2_UNDO`: stores Undo data of `NOLOGGING` tables.

## Tablespace Operations

### Creating a Tablespace

Users can use the `CREATE TABLESPACE` command to create a tablespace:

```sql
CREATE TABLESPACE TABLESPACE_NAME EXTENTS PAGE_NUM DATAFILE 'DATA_FILE_NAME' SIZE FILE_SIZE AUTOEXTEND ON NEXT EXTEND_SIZE;
```

- The `EXTENTS` parameter indicates the number of `PAGE`s contained in one `EXTENT`. The value range is [8,8192], and the value must be an integer power of 2. If this parameter is not specified, the default value is 8.

- `EXTENTS` is the basic unit by which a table requests space from the tablespace when storing row data. Increasing it appropriately can improve I/O performance, but it may cause space waste for small tables.

- `DATAFILE 'DATA_FILE_NAME' SIZE FILE_SIZE` indicates the name of the specified data file, whose initial file size is `FILE_SIZE`.

- `AUTOEXTEND ON NEXT EXTEND_SIZE` sets automatic extension, that is, the tablespace automatically extends by `EXTEND_SIZE` after the data fills up `FILE_SIZE`.

Example:

Create a tablespace named `OGRACSPACE`, with one `EXTENT` containing 16 `PAGE`s, specify its data file as `OGRACSPACE1`, set the initial size of the data file to `128M`, and enable automatic extension of `64M`.

```sql
CREATE TABLESPACE OGRACSPACE EXTENTS 16 DATAFILE 'OGRACSPACE1' SIZE 128M AUTOEXTEND ON NEXT 64M;
-- Create a table structure on the tablespace and insert data
CREATE TABLE DEPT(f1 INT, f2 INT, f3 INT) TABLESPACE OGRACSPACE;
INSERT INTO DEPT VALUES(1, 2, 3),(2, 3, 4),(null, 4, 5);
COMMIT;
```

### Viewing Tablespace Information

DBA can view tablespace-related information through `ADM_TABLESPACES` and `ADM_SEGMENTS`. Examples are as follows:

```sql
SQL> SELECT PAGE_SIZE, EXTENT_PAGES, DATAFILE_COUNT, TOTAL_SIZE, USED_SIZE FROM ADM_TABLESPACES WHERE TABLESPACE_NAME = 'OGRACSPACE';
PAGE_SIZE            EXTENT_PAGES DATAFILE_COUNT TOTAL_SIZE           USED_SIZE           
-------------------- ------------ -------------- -------------------- --------------------
8192                 16           1              134217728            147456              
                                               
SQL> SELECT SEGMENT_NAME, SEGMENT_TYPE, PAGES, EXTENTS FROM ADM_SEGMENTS WHERE TABLESPACE_NAME = 'OGRACSPACE';
SEGMENT_NAME                                                     SEGMENT_TYPE PAGES                EXTENTS             
---------------------------------------------------------------- ------------ -------------------- --------------------
DEPT                                                             TABLE        16                   1                   
```

### Modifying a Tablespace

Users can use the `ALTER TABLESPACE` command to rename a tablespace, or extend or shrink a tablespace.

```sql
-- Rename a tablespace
ALTER TABLESPACE TABLESPACE_NAME RENAME TO TABLESPACE_NAME_NEW;
-- Extend a tablespace by adding a DATAFILE
ALTER TABLESPACE TABLESPACE_NAME ADD DATAFILE 'FILE_NAME' SIZE FILE_SIZE;
-- Drop a DATAFILE from a tablespace
ALTER TABLESPACE TABLESPACE_NAME DROP DATAFILE 'FILE_NAME';
-- Enable or disable AUTOEXTEND
ALTER TABLESPACE TABLESPACE_NAME AUTOEXTEND OFF[ON];
-- Shrink the tablespace
ALTER TABLESPACE TABLESPACE_NAME SHRINK SPACE KEEP FILE_SIZE;
```

Example:

```sql
-- Rename the tablespace
ALTER TABLESPACE OGRACSPACE RENAME TO OGRACSPACENEW;
-- Extend the tablespace by adding a DATAFILE
ALTER TABLESPACE OGRACSPACE ADD DATAFILE 'OGRACSPACE2' SIZE 128M;
-- Drop a DATAFILE from a tablespace
ALTER TABLESPACE OGRACSPACE DROP DATAFILE 'OGRACSPACE2';
-- Enable or disable AUTOEXTEND
ALTER TABLESPACE OGRACSPACE AUTOEXTEND OFF[ON];
-- Shrink the tablespace
ALTER TABLESPACE OGRACSPACE SHRINK SPACE KEEP 200M;
```

### Deleting a Tablespace

Users can use the `DROP TABLESPACE` command to delete a manually created tablespace. The `SYSTEM`, `TEMP`, `UNDO`, `USERS`, and `SYSAUX` tablespaces cannot be deleted.

Before deleting a tablespace, check its usage, clear the data in the tablespace, and finally execute the `DROP TABLESPACE` operation.

```sql
SELECT * FROM ADM_TABLESPACES WHERE TABLESPACE_NAME = 'OGRACSPACE';
SELECT * FROM ADM_SEGMENTS WHERE TABLESPACE_NAME = 'OGRACSPACE';
```

```sql
-- Delete the table
DROP TABLE TABLE_NAME;
-- Purge the tablespace
PURGE TABLESPACE TABLESPACE_NAME;
-- Delete the tablespace
DROP TABLESPACE TABLESPACE_NAME;
```
