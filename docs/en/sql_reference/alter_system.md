# ALTER SYSTEM

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:08.762Z pushedAt=2026-08-27T08:54:46.169Z -->

## Function Description

Modifies database system parameters or performs specific system-level operations.

## Precautions

The `ALTER SYSTEM` system permission is required to execute this command.

## Syntax

ALTER SYSTEM

 { DUMP DATAFILE file_id PAGE page_id

 | SWITCH LOGFILE

 | SET parameter_name = parameter_value [ SCOPE = { MEMORY | PFILE | BOTH } ]

 | LOAD DICTIONARY FOR [ schema_name.]object_name

 | INIT DICTIONARY | RELOAD {HBA | PBL} CONFIG

 | REFRESH SYSDBA PRIVILEGE

 | KILL SESSION 'session_id,serial'

 | RESET STATISTIC | CHECKPOINT

 |{ ADD | DELETE } LSNR_ADDR LISTENING_IP

 |{ ADD | DELETE } HBA ENTRY hba_conf_entry

 | FLUSH {BUFFER | SQLPOOL}

 | DEBUG MODE debug_parameter_name = debug_parameter_value

 | DUMP CATALOG { TABLE table_name | USER user_name } [ TO 'folders' ]

 | RECYCLE SHAREDPOOL [FORCE]

 | REPAIR CATALOG }

## Parameter Description

- `DUMP DATAFILE file_id PAGE page_id`

Exports a specific page of the specified data file.

- `file_id`

Specifies the file ID, with a value range of [0, 2147483648). It can be obtained by querying `FILE_ID` in the `ADM_DATA_FILES` DBA view.

- `page_id`

Specifies the page ID, with a value range of [1, 2147483648). It must be a positive integer and less than the high water mark of the number of pages used by the file.

The high water mark can be obtained by querying `HIGH_WATER_MARK` in the dynamic performance view `DV_DATA_FILES`.

- `SWITCH LOGFILE`

Switches the current log file.

- `SET parameter_name = parameter_value [ SCOPE = { MEMORY | PFILE | BOTH } ]`

Modifies system parameters. If `SCOPE` is set to `PFILE` or `BOTH`, parameter settings are written into the `Zenith.ini` configuration file. The values of `SCOPE` are defined as follows:  
• `MEMORY`: Parameter settings take effect only in memory and become invalid after a restart. This option is applicable only to dynamic parameters.  
• `PFILE`: Parameter settings are written into the configuration file only and take effect upon the next startup. Both dynamic and static parameters are supported. This is the only option applicable to static parameters.  
• `BOTH` (default): Parameter settings are written into both the initialization parameter file and memory and take effect immediately. This option is applicable only to dynamic parameters.

- `LOAD DICTIONARY FOR [schema_name].object_name`: Loads an object into the data dictionary.

- `INIT DICTIONARY`

Loads entries (such as system views, dynamic views, sequences, and roles) except the system catalogs.

The prerequisites are that the database is in the `RESTRICTED` mode and all system catalogs are loaded by running `ALTER SYSTEM LOAD DICTIONARY FOR [schema_name].object_name;`.

- `RELOAD {HBA | PBL} CONFIG`

Loads the `oghba.conf` file online to make the trustlist configuration take effect.

Loads the `pbl.conf` file online to make the weak password configuration take effect.

- `REFRESH SYSDBA PRIVILEGE`

Updates the ciphertext and encryption key used for password-free login of the `SYSDBA` user online. The update does not affect the connected clients. The new key is used for new connections.

- `KILL SESSION 'session_id,serial'`

Terminates a session. `session_id` specifies the session ID, and `serial` specifies the serial number.

- `RESET STATISTIC`

Clears the statistics in the dynamic view `DV_SYS_STATS`.

- `CHECKPOINT`

Executes checkpoints for the current instance to ensure that all changes made to the committed transactions are written to the disk.

- `{ ADD | DELETE } LSNR_ADDR LISTENING_IP`

Adds/Deletes the listening IP address (must be enclosed in quotation marks). Currently, up to eight listening IP addresses are supported.

If you add an IP address that does not correspond to any existing NIC as a floating listening IP address, an error is returned directly.

**NOTE**: Deleting an IP address that is currently in use will cause related connections to be interrupted and transactions to be rolled back.

- `{ ADD | DELETE } HBA ENTRY hba_conf_entry`

Adds a user trustlist entry to or deletes a user trustlist entry from the `oghba.conf` file.

- `hba_conf_entry`

The entry format is `'type user address'`. The parameters are described as follows:

  - `type`: Specifies the connection type.

    - `host`: Indicates a common TCP or SSL connection.

    - `hostssl`: Indicates an SSL connection only (if SSL is enabled on the server but disabled on the client, the server will reject the connection request).

  - `user`: Specifies the user allowed to access the database. `*` indicates all users. If the username contains special characters (such as `#`, `*`, and tab), it must be enclosed in double quotation marks (""), for example, `"#abc"`. Only one user can be specified per line.

  - `address`: Specifies the IP address range allowed for access (multiple addresses can be separated by commas). The IP addresses can be IPv4 or IPv6 addresses, or a network segment with the subnet mask or prefix length specified. Supported formats:

    - **A single IPv4/IPv6 address**: `192.168.1.111` or `20AB::9217:acff:feab:fcd0`

    - **Subnet mask**: `192.168.2.0/24` or `20CD::2654:addf:3ab2:fed0/64`

    - **All network segments**: `*.*.*.*` or `0.0.0.0/0`

- `FLUSH BUFFER`

Clears the database buffer.

- `FLUSH SQLPOOL`

Clears the SQL pool buffer.

- `DEBUG MODE debug_parameter_name = debug_parameter_value`

Specifies the debugging parameter. The setting takes effect immediately. All debugging parameters are not written into the configuration file and are stored only in the memory. The setting is restored after a system restart.

Debugging parameters are used only for development and debugging. Do not modify them; otherwise, database exceptions may occur.

- `DUMP CATALOG TABLE table_name`

Dumps the data dictionary memory information and associated index information of a specified table.

- `DUMP CATALOG USER user_name`

Dumps the data dictionary memory information of a specified user.

- `[ TO 'folders']`

Specifies the output directory of dump data (default: the `trc` folder).

The maximum size of each dump file is 10 MB. If the dump data exceeds this limit, an error is reported. In this case, you must manually delete the file and re-execute the `DUMP` operation.

The `SYS` user can dump information about all users. Common users can dump only their own information. A DBA can dump information about common users and other DBAs.

- `RECYCLE SHAREDPOOL [FORCE]`

Recycles DC/SQL pool to the shared area.

`FORCE` indicates that all soft parsing in the SQL pool is forcibly set to `FALSE`.

- `REPAIR CATALOG`

Repairs inconsistent column definitions of the core system catalogs (for example, binary records after an upgrade do not match the table structure in data).

## Examples

- Export a specified page of a data file.

```sql
-- Drop a tablespace.
DROP TABLESPACE IF EXISTS test_space;
```

```
-- Create a tablespace.
CREATE TABLESPACE video_space DATAFILE 'test_dfile1' SIZE 32M;
```

```
-- Query FILE_ID.
SELECT FILE_NAME,FILE_ID FROM ADM_DATA_FILES WHERE FILE_NAME='/opt/ograc/data/data/test_dfile1';
```

```
-- Query the high water mark of the data file.
SELECT * FROM DV_DATA_FILES WHERE FILE_NAME='/opt/ograc/data/data/test_dfile1';
```

```
-- Dump the first page of the data file (assuming FILE_ID=17).
ALTER SYSTEM DUMP datafile 17 PAGE 1;
```

- Switch the log file.

```
ALTER SYSTEM SWITCH LOGFILE;
```

- Change the value of `UNDO_RETENTION_TIME` to 1200s. The setting takes effect immediately only in the memory and becomes invalid after a restart.

```
-- Query the current value.
SHOW PARAMETER UNDO_RETENTION_TIME;
```

```
-- Change the value.
ALTER SYSTEM SET UNDO_RETENTION_TIME=1200 SCOPE=MEMORY;
```

- Load a table into the data dictionary.

```
-- Drop the table education.
DROP TABLE IF EXISTS education;
-- Create the table education.
CREATE TABLE education(staff_id INT, highest_degree CHAR(8) NOT NULL, graduate_school VARCHAR(64), graduate_date DATETIME, education_note VARCHAR(70));
-- Load the table education into the data dictionary.
ALTER SYSTEM LOAD DICTIONARY FOR education;
```

- Load entries (such as system views, dynamic views, sequences, and roles) except the system catalogs.

The prerequisites are that the database is in the `RESTRICTED` mode and all system catalogs are loaded by running `ALTER SYSTEM LOAD DICTIONARY FOR [schema_name].object_name`.

```
ALTER SYSTEM INIT DICTIONARY;
```

- Load the `oghba.conf` file online.

```
ALTER SYSTEM RELOAD HBA CONFIG;
```

- Add a listening IP address.

```
ALTER SYSTEM ADD LSNR_ADDR '10.10.10.11';
```

- Delete a listening IP address.

```sql
ALTER SYSTEM DELETE LSNR_ADDR '10.10.10.11';
```

- Update the ciphertext and encryption key for password-free login of the `SYSDBA` user.

```sql
ALTER SYSTEM REFRESH SYSDBA PRIVILEGE;
```

- Clear the statistics in the dynamic view `DV_SYS_STATS`.

```sql
ALTER SYSTEM RESET STATISTIC;
```

- Execute checkpoints for the current instance.

```sql
ALTER SYSTEM CHECKPOINT;
```

- Clear buffer.

  Clear the database buffer.

```sql
ALTER SYSTEM FLUSH BUFFER;
```

  Clear the SQL pool buffer.

```sql
ALTER SYSTEM FLUSH SQLPOOL;
```

- Modify a database debugging parameter.

```sql
ALTER SYSTEM DEBUG MODE _MRP_RES_LOGSIZE = 1G;
```

- Dump data dictionary information.

```
ALTER SYSTEM DUMP CATALOG TABLE TEST;
ALTER SYSTEM DUMP CATALOG USER TEST;
```
