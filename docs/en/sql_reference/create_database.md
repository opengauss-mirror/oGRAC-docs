# CREATE DATABASE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:14:36.393Z pushedAt=2026-08-13T08:33:52.610Z -->

## Function Description

Creates a new database.

## Precautions

- Only users with the CREATE DATABASE permission can create a new database. System administrators have this permission by default.

- It is automatically created during database installation and does not need to be created manually.

- If creation fails, restart the database and create it again.

## Syntax

```sql
CREATE DATABASE CLUSTERED database_name 
             [ { [ CHARACTER SET ] |
               [ CONTROLFILE('file1','file2','file3',....) ] |
               [ SYSTEM TABLESPACE DATAFILE ] |
               [ NOLOGGING TABLESPACE TEMPFILE ] |
               [ NOLOGGING UNDO TABLESPACE TEMPFILE ] |
               [ DEFAULT TABLESPACE DATAFILE ] |
               [ SYSAUX TABLESPACE DATAFILE ] |
               [ UNDO TABLESPACE DATAFILE ] |
               [ TEMPORARY TABLESPACE TEMPFILE ] |
               [ LOGFILE ]}[...] ]
               [ WITH DBCOMPATIBILITY 'compatibility_type'];
```

## Parameter Description

- **database_name**: Database name.

    Value range: a string that conforms to the identifier naming conventions.

- **character set**: Specifies the character encoding used by the database.

    Value range: only UTF-8 and GBK are supported. If not specified, the default encoding is UTF-8.

- **controlfile**: Control file list.

    Value range: a string, with file names separated by commas.

- **tablespace**: Tablespace-related parameters include: `system tablespace datafile`, `nologging tablespace TEMPFILE`, `nologging undo tablespace TEMPFILE`, `default tablespace datafile`, `sysaux tablespace DATAFILE`, `undo tablespace datafile`, and `temporary tablespace TEMPFILE`.

    > **NOTE**
    >- system tablespace datafile: the data file of the SYSTEM tablespace, used to store core data. Value range: 128M to 8T.
    >- nologging tablespace TEMPFILE: the NOLOGGING tablespace. Value range: 1M to 8T.
    >- nologging undo tablespace TEMPFILE: the NOLOGGING UNDO tablespace. Value range: 128M to 32G.
    >- default tablespace datafile: the default storage tablespace for user-created objects (such as tables and indexes). Value range: 1M to 8T.
    >- sysaux tablespace datafile: used to store data of other database components and tools beyond the data dictionary, reducing the burden on the SYSTEM tablespace. Value range: 128M to 8T.
    >- undo tablespace datafile: used to store transaction undo information. Value range: 128M to 32G.
    >- temporary tablespace TEMPFILE: used to create and manage temporary tablespaces. Temporary tablespaces are primarily used to store temporary data generated during database operations, such as sorting operations and hash joins that require a large amount of temporary storage space. Value range: 5M to 8T.

- **logfile**: Creates database log files. Parameters include: `logfile`, `size`, and `blocksize`.

    > **NOTE**
    >- logfile: log file.
    >- size: log file size. The default unit is byte. Other options include K, M, G, T, P, and E. At least three log files are required, and size >= 56M + 16K + log_buffer_size must be satisfied.
    >- blocksize: block size in bytes. Only 512 or 4096 is supported.

- **compatibility_type**: compatibility of the database to be created. A, B, and C compatibility are supported. If `with dbcompatibility` is not used to specify the compatibility, the database defaults to an A-compatible database.

## Examples

```
create database clustered ograc
    character set utf8
    controlfile('dbfiles1/ctrl1', 'dbfiles1/ctrl2', 'dbfiles1/ctrl3')
    system tablespace datafile 'dbfiles1/sys.dat' size 128M autoextend on next 32M
    nologging tablespace TEMPFILE 'dbfiles1/temp2_01' size 160M autoextend on next 32M, 'dbfiles1/temp2_02' size 160M autoextend on next 32M
    nologging undo tablespace TEMPFILE 'dbfiles1/temp2_undo' size 1G
    default tablespace datafile 'dbfiles1/user1.dat' size 1G autoextend on next 32M, 'dbfiles1/user2.dat' size 1G autoextend on next 32M
    sysaux tablespace DATAFILE 'dbfiles1/sysaux' size 160M autoextend on next 32M
    undo tablespace datafile 'dbfiles1/undo01.dat' size 1G autoextend on next 32M, 'dbfiles1/undo02.dat' size 1G autoextend on next 32M
    temporary tablespace TEMPFILE 'dbfiles1/temp1_01' size 160M autoextend on next 32M, 'dbfiles1/temp1_02' size 160M autoextend on next 32M
    nologging undo tablespace TEMPFILE 'dbfiles1/temp2_undo_01' size 128M autoextend on next 32M
    logfile ('dbfiles1/redo01.dat' size 256M blocksize 512, 'dbfiles1/redo02.dat' size 256M blocksize 4096, 'dbfiles1/redo03.dat' size 256M blocksize 512);
```
