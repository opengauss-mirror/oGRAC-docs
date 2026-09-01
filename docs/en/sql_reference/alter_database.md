# ALTER DATABASE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:04:10.352Z pushedAt=2026-08-25T01:41:48.907Z -->

## Function Description

Modifies a database.

## Precautions

- Only users granted the `ALTER DATABASE` permission can execute this command.

## Syntax

- Specify the database state as `MOUNT` or `OPEN`.

    ```
    ALTER DATABASE [ database_name ] {
        MOUNT
        | OPEN [ RESETLOGS
                 | READ ONLY
                 | READ WRITE
                 | RESTRICTED
                 | UPGRADE
                 | FORCE IGNORE LOGS
                 | [ UPGRADE ] REPLAY UNTIL lfn ]
               [ IGNORE SYSTIME ]
        };
    ```

- Add or delete log files.

    ```
    ALTER DATABASE [ database_name ] {
        ARCHIVELOG
        | NOARCHIVELOG
        | ADD LOGFILE (
            { 'file_name' SIZE integer [ B | K | M | G | T | P | E ]
              [ BLOCKSIZE { 512 | 4096 } ] }
            [,...] ) 
        | DROP LOGFILE ( 'file_name' )
        | ARCHIVE LOGFILE ( { 'file_name' } [,...] ) }
    ```

- Delete archive log files.

    ```
    ALTER DATABASE [ database_name ] DELETE ARCHIVELOG { ALL | UNTIL TIME 'date_string' } [ FORCE ]
    ```

- Set the data protection mode on the standby database.

    ```
    ALTER DATABASE [ database_name ] SET STANDBY DATABASE TO MAXIMIZE { PROTECTION | AVAILABILITY | PERFORMANCE }
    ```

- Modify the attributes of one or more data files.

    ```
    ALTER DATABASE [ database_name ] DATAFILE
        { 'file_name' | file_number } [,...]
        { AUTOEXTEND { OFF
                       | ON [ NEXT integer [ K | M | G ]
                              | MAXSIZE { integer [ K | M | G ] | UNLIMITED }
                            ]
                     }
          | RESIZE integer [ K | M | G ] }
    ```

- Clear and rebuild a log file.

    ```
    ALTER DATABASE [ database_name ] CLEAR LOGFILE file_id
    ```

- Perform primary/standby switchover.

    ```
    ALTER DATABASE [ database_name ] SWITCHOVER [TIMEOUT tm_s]
    ```

- Switch the standby database to primary.

    ```
    ALTER DATABASE [ database_name ] FAILOVER [ FORCE ]
    ```

- Cancel the upgrade mode.

    ```
    ALTER DATABASE [ database_name ] CANCEL UPGRADE
    ```

- Modify the database state.

    ```
    ALTER DATABASE [ database_name ] CONVERT TO 
        { READONLY
          | READWRITE
          | [ CASCADED ] PHYSICAL STANDBY [ MOUNT ] }
    ```

- Delete the physical files of a backup set and the backup set records in `SYS_BACKUP_SETS`.

    ```
    ALTER DATABASE [ database_name ] DELETE BACKUPSET 'tag' [ FORCE ]
    ```

- Rebuild a tablespace.

    ```
    ALTER DATABASE [ database_name ] REBUILD TABLESPACE tablespace_name
    ```

- Enable or disable global logical replication.

    ```
    ALTER DATABASE [ database_name ] ENABLE_LOGIC_REPLICATION { ON | OFF }
    ```

- Update and synchronize the master key.

    ```
    ALTER DATABASE [ database_name ] UPDATE MASTERKEY
    ```

## Parameter Description

- `database_name`: Indicates the name of the database to be modified. If the database name is not specified, the database currently in the `MOUNT` state is used.

- `MOUNT`: Indicates that the database is mounted but not started.

- `OPEN`: Indicates that the database is started normally. It has the following sub-options:

    - `RESETLOGS`: Increments `RSTID` by 1.

    - `READ ONLY`: Indicates the read-only mode, where the database can only be queried.

    - `READ WRITE`: Indicates the read-write mode, which is the default mode when a database becomes `OPEN`.

    - `RESTRICTED`: Indicates the restricted mode, which is used to support DFX capabilities such as database maintenance and emergency repair. In this mode, only the `SYS` user can start the database to perform operations, and only one session connection is supported.

    - `UPGRADE`: Indicates the upgrade mode, where only the core system catalogs are loaded. In this mode, only the `SYS` user can start the database to perform operations, and only one session connection is supported.

    - `FORCE IGNORE LOGS`: Forcefully ignores log files.

    - `REPLAY UNTIL lfn`: Indicates that the standby instance is started in this mode and stops replaying when it reaches the _lfn_ point.

    - `IGNORE SYSTIME`: Ignores the impact of system time jumps on the database.

- `ARCHIVELOG`: Enables redo log archiving.

- `NOARCHIVELOG`: Disables redo log archiving.

- `ADD LOGFILE ( { 'file_name' SIZE integer [ B | K | M | G | T | P | E ] [ BLOCKSIZE { 512 | 4096 } ] } [,...] )`: Adds one or more redo log files to the primary node.

    - `file_name`: Indicates the file name.

    - `SIZE integer [ B | K | M | G | T | P | E ]`: Indicates the file size. The default unit is bytes. `B` indicates bytes, `K` indicates KB, `M` indicates MB, `G` indicates GB, `T` indicates TB, `P` indicates PB, and `E` indicates EB.

    - `BLOCKSIZE { 512 | 4096 }`: Indicates the file block size, in bytes. The value can be `512` or `4096`, and the default value is `512` bytes.

- `DROP LOGFILE ( 'file_name' )`: Deletes a redo log file. Only one can be deleted at a time.

- `ARCHIVE LOGFILE ( { 'file_name' } [,...] )`: Archives online logs. One or more can be archived.

- `DELETE ARCHIVELOG { ALL | UNTIL TIME 'date_string' } [ FORCE ]`: Deletes archive log files. This can only be performed in archive mode.

    - `ALL`: Deletes all archive logs that meet the following three conditions:

        - Archive logs are not required by the local recovery. That is, the IDs of the archive logs are smaller than the log IDs in local `rcy_point`.

        - Archive logs have been backed up. (You can ignore the backup by setting the `FORCE` parameter.)

        - The archive log number is smaller than that of the consecutively archived log on the standby node.

    - `UNTIL TIME 'date_string'`: Deletes the archive logs that meet the deletion conditions and are generated before the specified date. The time format is _YYYY-MM-DD hh:mm:ss_.

    - `FORCE`: Deletes an archive log no matter whether it has been backed up or not.

- `SET STANDBY DATABASE TO MAXIMIZE { PROTECTION | AVAILABILITY | PERFORMANCE }`: Specifies the data protection mode on the standby database node.

    - `PROTECTION`: Indicates the maximum protection mode, which provides the highest level of data protection. Transactions on the primary database can be committed only after the standby database receives the redo logs.

    - `AVAILABILITY`: Indicates the maximum availability mode. If redo logs cannot be written to the standby database, the data protection mode is temporarily changed to `PERFORMANCE` until the standby database receives the redo logs.

    - `PERFORMANCE`: Indicates the maximum performance mode, which ensures the highest availability of the primary database. Transactions on the primary database are committed no matter whether the standby database receives the redo logs. However, if the standby database fails to receive the redo logs related to the transactions committed by the primary database, the transaction data will be lost.

- `DATAFILE { 'file_name' | file_number } [,...] { AUTOEXTEND { OFF | ON [ NEXT integer [ K | M | G ] | MAXSIZE { integer [ K | M | G ] | UNLIMITED } ] } | RESIZE integer [ K | M | G ] }`: Modifies the attributes of one or more data files. Data files can be specified by file name or file number.

    - `file_name`: Indicates the file name. The value can be an absolute path or a file name. If the latter is used, the database generates a full path based on the file name and the `data` directory in the specified database instance path.

    - `file_number`: Indicates the file number, that is, the ID of a data file in the database.

    - `AUTOEXTEND`: Enables or disables automatic extension, and specifies the extension size and upper limit.

        - `OFF`: Disables automatic extension.

        - `ON`: Enables automatic extension.

            - `NEXT`: Specifies the extension size. The default value is 16 MB.

            - `MAXSIZE`: Specifies the extension upper limit, which cannot exceed the current file size.

            - `UNLIMITED`: Indicates automatic extension without an upper limit.

    - `RESIZE`: Modifies the size of a data file.

- `CLEAR LOGFILE file_id`: Clears and rebuilds the log file specified by `file_id`. If the database cannot be started due to damage of a log file header and the log file can be cleared, you can use this clause to clear and rebuild the log file so that the database can be started.

- `SWITCHOVER [TIMEOUT tm_s]`: Performs a switchover between the primary and standby databases. The timeout interval `tm_s` is an integer, in seconds. Its value can be `0` or an integer in the range [30, 1800]. The default value is `0`.

- `FAILOVER [ FORCE ]`: Promotes the standby database to primary when the primary/standby relationship is abnormal. If `FORCE` is specified, this operation can be performed regardless of whether the primary/standby relationship is abnormal.

- `CANCEL UPGRADE`: Cancels the `UPGRADE` mode. After the database is upgraded, you need to cancel the `UPGRADE` mode.

- `CONVERT TO { READONLY | READWRITE | [ CASCADED ] PHYSICAL STANDBY [ MOUNT ] }`: Modifies the database role or state.

    - `READONLY`: Indicates the read-only mode, where the database can only be queried.

    - `READWRITE`: Enables read and write. It is the default state after the database becomes `OPEN`.

    - `[ CASCADED ] PHYSICAL STANDBY [ MOUNT ]`: Changes the database role to standby or cascaded standby. If `MOUNT` is specified, only the role is changed and the database state remains unchanged. If `MOUNT` is not specified, the database automatically changes to `READ ONLY` under the `OPEN` state after the role is changed.

- `DELETE BACKUPSET 'tag' [ FORCE ]`: Deletes the physical files of the backup set and the backup set records in `SYS_BACKUP_SETS`. If the backup set does not exist on the disk or the backup set media is not a disk, an error is reported. For an incremental backup set, if other backup sets in the system catalog depend on the backup set, an error is reported. In this case, delete the backup sets that depend on the backup set.

    - `FORCE`: Forcibly deletes the backup set records from `SYS_BACKUP_SETS`.

- `REBUILD TABLESPACE tablespace_name`: Rebuilds a tablespace.

- `ENABLE_LOGIC_REPLICATION { ON | OFF }`: Enables or disables global logical replication.

- `UPDATE MASTERKEY`: Updates and synchronizes the master key. This clause applies to the tablespace TDE function.

## Examples

```
-- Change the database state to MOUNT.
SQL> ALTER DATABASE MOUNT;

-- Change the database state from MOUNT to OPEN.
SQL> ALTER DATABASE OPEN;

-- Reset the database log sequence number to 1 in the MOUNT database state.
SQL> ALTER DATABASE RESETLOGS;

-- Change the database state from MOUNT to READ ONLY.
SQL> ALTER DATABASE OPEN READ ONLY;

-- Add a redo log file named test1 with a size of 1 GB and a block size of 4,096 bytes.
SQL> ALTER DATABASE ADD LOGFILE ('test1' SIZE 1G BLOCKSIZE 4096);

-- Delete the redo log file named test1.
SQL> ALTER DATABASE DROP LOGFILE ('test1');

-- Delete archive logs that meet the conditions and were generated before 2025/11/15 11:00:00.
SQL> ALTER DATABASE DELETE ARCHIVELOG UNTIL TIME '2025/11/15 11:00:00';

-- Disable automatic extension for the data file whose file ID is 1.
SQL> ALTER DATABASE DATAFILE 1 AUTOEXTEND OFF;

-- Change the extension size of the data file whose file ID is 1 to 20 MB.
SQL> ALTER DATABASE DATAFILE 1 AUTOEXTEND ON NEXT 20M;

-- Change the extension upper limit of the data file whose file ID is 1 to 10 GB.
SQL> ALTER DATABASE DATAFILE 1 AUTOEXTEND ON MAXSIZE 10G;

-- Change the extension upper limit of the data file whose file ID is 1 to UNLIMITED.
SQL> ALTER DATABASE DATAFILE 1 AUTOEXTEND ON MAXSIZE UNLIMITED;

-- Change the database role to standby in the MOUNT database state.
SQL> ALTER DATABASE CONVERT TO PHYSICAL STANDBY;

-- Change the database state to READWRITE.
SQL> ALTER DATABASE CONVERT TO READWRITE;

-- Change the size of the USER1 data file to 128 MB.
SQL> ALTER DATABASE DATAFILE 'USER1' RESIZE 128M;

-- Rebuild the header of the log file whose file ID is 0 in the MOUNT database state.
SQL> ALTER DATABASE CLEAR LOGFILE 0;

-- Delete the files and records of the backup set whose tag is incr_bak.
SQL> ALTER DATABASE DELETE BACKUPSET 'incr_bak';

-- Enable global logical replication.
SQL> ALTER DATABASE ENABLE_LOGIC_REPLICATION ON;
```