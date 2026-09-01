# Database Parameter Description

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:01:03.120Z pushedAt=2026-08-24T07:02:10.624Z -->

## Audit and Security

### AUDIT_LEVEL

**Parameter Description**: Database audit level, which uses a bitmask mechanism to combine audit scopes by performing a "bitwise OR" operation on the values corresponding to different operation types. For example, setting it to `5` means auditing DDL and DML types.  
**Value Range**: [0, 255]  
**Options**:

- `0`: Disables auditing.

- `1`: DDL (Data Definition Language)

- `2`: DCL (Data Control Language)

- `4`: DML (Data Manipulation Language)

- `8`: PL (Procedural Language)

- `255`: ALL (All Types)

**Default Value**: `3`, with an installation default of `0`

### ENABLE_SYSDBA_LOGIN

**Parameter Description**: Sets whether to support password-free SYSDBA login (modifiable only by the sys user).  
**Value Range**: `FALSE`, `TRUE`  
**Default Value**: `TRUE`. Modification takes effect after restart.

### ENABLE_ACCESS_DC

**Parameter Description**: Controls whether the SELECT ANY TABLE or READ ANY TABLE permission can access SYS objects.  
**Value Range**: `FALSE`, `TRUE`  
**Default Value**: `TRUE`, with an installation default of `FALSE`

### REPLACE_PASSWORD_VERIFY

**Parameter Description**: Specifies whether the old password is required when a regular user changes their own password.  
**Value Range**: `FALSE`, `TRUE`  
**Default Value**: `FALSE`, with an installation default of `TRUE`

### _SYS_PASSWORD

**Parameter Description**: The ciphertext of the SYS user password encrypted by an irreversible encryption algorithm. If modification is required, it is recommended to use `alter user` to change it.  
**Value Range**: String, maximum length 1023  
**Default Value**: The encrypted value of the sys password entered during database installation

## Storage and Performance

### PAGE_CHECKSUM

**Parameter Description**: Whether to enable PAGE checksum verification. This slightly increases CPU load but improves data security.  
**Value Range**:

- `OFF`: Disables verification.

- `TYPICAL`: Verifies PAGE when it is swapped between memory and disk.

- `FULL`: Compared with `TYPICAL`, also verifies PAGE before it is modified in memory.

**Default Value**: `TYPICAL`. Modification takes effect after restart.

### _AUTO_INDEX_RECYCLE

**Parameter Description**: Whether to recycle index empty pages in the background. When shared storage is used, ensure that all instances have consistent settings.  
**Value Range**: `ON`, `OFF`  
**Default Value**: `ON`

### DEFAULT_EXTENTS

**Parameter Description**: The number of pages contained in one EXTENT by default. A smaller value causes frequent EXTENT allocation under high pressure and lower performance in read-ahead scenarios; a larger value delivers better performance but wastes space noticeably for small tables.  
**Value Range**: `8`, `16`, `32`, `64`, `128`  
**Default Value**: `8`, with an installation default of `128`

### TEMP_POOL_NUM

**Parameter Description**: Number of TEMP_POOLs. Each session selects one TEMP_POOL to allocate VM pages at startup.  
**Value Range**: [1, 128]  
**Default Value**: `1`, with an installation default of `8`. Modification takes effect after restart.

## Transaction and Lock Management

### CR_MODE

**Parameter Description**: CR is the abbreviation of consistent read. During concurrent access, the current page may be invisible to the current session, and a visible page needs to be constructed based on transaction principles, which is called a consistent read page. This parameter is used to set the MVCC mechanism for tables/indexes.  
**Value Range**:

- `PAGE`: page-level MVCC

**Default Value**: `PAGE`. Modification takes effect after restart.

### CR_POOL_SIZE

**Parameter Description**: Size of the consistent read page buffer. A larger value accelerates concurrent access.  
**Value Range**: [16M, 32T], in bytes.  
**Default Value**: `32M`, with an installation default of `1G`. Modification takes effect after restart.

### CR_POOL_COUNT

**Parameter Description**: Number of consistent read page buffers. A larger value can reduce lock contention among different sessions.  
**Value Range**: [1, 256]  
**Default Value**: `1`, with an installation default of `32`. Modification takes effect after restart.

### _PRIVATE_KEY_LOCKS

**Parameter Description**: Maximum number of key locks retained per session when a transaction ends. The remaining key locks are released and returned to the global lock area, and are reused when a new transaction starts.  
**Value Range**: [8, 128]  
**Default Value**: `8`, with an installation default of `128`. Modification takes effect after restart.

### _PRIVATE_ROW_LOCKS

**Parameter Description**: The maximum number of row locks retained per session when a transaction ends. The remaining row locks are released and returned to the global lock area, and are reused when a new transaction starts.  
**Value Range**: [8, 128]  
**Default Value**: `8`, with an installation default of `128`. Modification takes effect after restart.

### UNDO_RETENTION_TIME

**Parameter Description**: UNDO retention time. If the retention time is too short, queries may report the error "snapshot too old". Set it as small as possible while satisfying the maximum query requirement. Setting it too large will increase the UNDO tablespace usage, and in high-concurrency scenarios may cause insufficient data buffer, leading to a surge in CPU usage. Therefore, careful evaluation is required.  
**Value Range**: (0, 4294967295], in seconds.  
**Default Value**: `100`, with an installation default of `600`

### _UNDO_SEGMENTS

**Parameter Description**: Number of UNDO segments, which determines the concurrency capability and the total number of transactions. Set this parameter only when creating a database; do not modify it in other cases.  
**Value Range**: (1, 1024]  
**Default Value**: `32`, with an installation default of `1024`

### _UNDO_ACTIVE_SEGMENTS

**Parameter Description**: The number of currently available and active UNDO segments. The configured value must be less than `_UNDO_SEGMENTS`.  
**Value Range**: (1, 1024]  
**Default Value**: `32`, with an installation default of `64`

## Memory Management

### VARIANT_MEMORY_AREA_SIZE

**Parameter Description**: Size of the memory area (VMA) used to store bind parameters whose execution size is less than 16 KB, with a page size of 16 KB. During database upgrade, it is recommended to set this parameter to `VARIANT_AREA_SIZE * SESSIONS * 0.8`. If there are many bind parameters and frequent execution, you may increase this value appropriately.  
**Value Range**: [4M, 32T]  
**Default Value**: `32M`, with an installation default of `2G`. Modification takes effect after restart.

### _VMP_CACHES_EACH_SESSION

**Parameter Description**: Number of 16 KB VMA memory pages that can be cached per session. When the available VMA memory is less than 10%, caching is not performed. If `_VMP_CACHES_EACH_SESSION * SESSIONS * 16K` is far greater than `VARIANT_MEMORY_AREA_SIZE`, it can be appropriately lowered.  
**Value Range**: [0, 4294967295]  
**Default Value**: `50`

### USE_LARGE_PAGES

**Parameter Description**: Usage mode of large pages in SGA memory  
**Value Range**:

- `TRUE`: Enables large pages, which requires that the large page feature is configured on the system.

- `FALSE`: Disables large pages (not recommended, as it may cause severe performance degradation).

**Default Value**: `TRUE`, with an installation default of `FALSE`. Modification takes effect after restart.

### LARGE_POOL_SIZE

**Parameter Description**: Size of the Large Pool, mainly used for storing SQL files and JSON syntax parsing data.  
**Value Range**: [4M, 32T], in bytes.  
**Default Value**: `32M`, with an installation default of `1G`. Modification takes effect after restart.

## High Availability and Recovery

### REPLAY_PRELOAD_PROCESSES

**Parameter Description**: Number of page preloading threads during REDO replay in the database recovery phase. A larger value improves concurrency efficiency but consumes more resources. It is recommended to set it to the same value as `LOG_REPLAY_PROCESSES`.  
**Value Range**: [0, 128]  
**Default Value**: `0`, which means no preloading. Modification takes effect after restart.

### LOG_REPLAY_PROCESSES

**Parameter Description**: Number of REDO replay threads during the database recovery phase. A larger value improves concurrency efficiency but consumes more resources.  
**Value Range**: [1, 128]  
**Default Value**: `1`, with an installation default of `64`. Modification takes effect after restart.

### _LOG_MAX_FILE_SIZE

**Parameter Description**: Maximum size of a single log file. If the log file size exceeds this parameter value, older log files may be deleted or backed up to new files.    
**Value Range**: [1M, 4G], in bytes.  
**Default Value**: `10M`

### _LOG_BACKUP_FILE_COUNT

**Parameter Description**: Maximum number of backup log files. When this number is exceeded, the earliest backup file is deleted to maintain a fixed count.  
**Value Range**: [0, 128]

- `0`: Disables log file backup. When a log file exceeds the size limit, it is deleted.

- Greater than 0: Enables log backup. When a log file exceeds the size limit, it is renamed to "old_log_file_name_current_timestamp.log".

**Default Value**: `10`

### RECYCLEBIN

**Parameter Description**: Whether to enable the recycle bin. The FLASHBACK feature can be used only after the recycle bin is enabled.  
**Value Range**: `FALSE`, `TRUE`  
**Default Value**: `FALSE`. Modification takes effect after restart.

## Jobs and Scheduling

### JOB_THREADS

**Parameter Description**: Alias for `JOB_QUEUE_PROCESSES`, the maximum number of concurrently executing jobs. Jobs exceeding the concurrency limit wait until the number of concurrent jobs is less than `JOB_THREADS` before execution.  
**Value Range**: [0, 200]  
**Default Value**: `100`

## Table Structure and Constraints

### MAX_COLUMN_COUNT

**Parameter Description**: Maximum number of columns supported by a table. This value cannot be decreased.  
**Value Range**: `1024`, `2048`, `3072`, `4096`  
**Default Value**: `1024`, with an installation default of `4096`. Modification takes effect after restart.

### ENABLE_IDX_KEY_LEN_CHECK

**Parameter Description**: Whether to check the index length when creating an index.  
**Value Range**:

- `TRUE`: Checks the index length when creating an index.

- `FALSE`: Index length is not checked at creation time, but is checked when being inserted into the database.

**Default Value**: `TRUE`

## Instance and Network Configuration

### INSTANCE_NAME

**Parameter Description**: Database instance name. After the database installation is complete, an instance name is automatically set for each instance.  
**Value Range**: String, with a maximum length of 63 bytes.  
**Default Value**: `oGRAC`. Modification takes effect after restart.

### INSTANCE_ID

**Parameter Description**: Database instance ID, which must be unique within the cluster.  
**Value Range**: [0, 63]  
**Default Value**: `0`. Modification takes effect after restart.

### LSNR_PORT

**Parameter Description**: Port on which the database listens.  
**Value Range**: [1024, 65535]  
**Default Value**: `1611`. Modification takes effect after restart.

### LSNR_ADDR

**Parameter Description**: Address or domain name that the database listens on, with multiple addresses comma-separated.  
**Value Range**: Valid IP address or domain name.  
**Default Value**: `127.0.0.1`. Modification takes effect after restart.

### INTERCONNECT_PORT

**Parameter Description**: Port listened on by the intra-cluster communication service (MES), with multiple ports comma-separated.  
**Value Range**: [1024, 65535]  
**Default Value**: `1611`. Modification takes effect after restart.

### INTERCONNECT_ADDR

**Parameter Description**: Address or domain name that the intra-cluster communication service listens on. Multiple addresses within the same instance are separated by commas; multiple instances are separated by semicolons. Example: `node0_ip1,node0_ip2;node1_ip1,node1_ip2`  
**Value Range**: Valid IP address or domain name  
**Default Value**: `127.0.0.1`. Modification takes effect after restart.

### INTERCONNECT_TYPE

**Parameter Description**: Intra-cluster communication protocol type (cannot be modified through SQL)  
**Value Range**:

- `TCP`: TCP communication

- `UC`: Unified communication protocol, using shared memory for inter-process communication within a node and TCP communication between nodes

- `UC_RDMA`: Unified communication protocol, using RDMA for inter-node communication

**Default**: `TCP`, with an installation default of `UC`

### INTERCONNECT_BY_PROFILE

**Parameter Description**: Whether to establish a connection when the intra-cluster communication service starts.  
**Value Range**: `FALSE`, `TRUE`  
**Default Value**: `FALSE`. Modification takes effect after restart.

## Directory and Path

### LOG_HOME

**Parameter Description**: Root directory where database log files are stored.  
**Value Range**: String, with a maximum length of 163.  
**Default Value**: `data_directory/log`. Modification takes effect after restart.

### SHARED_PATH

**Parameter Description**: Specifies the database data storage mode, which is set during installation.  
**Value Range**:

- `-`: Uses dbstor to store data.

- `+storage volume group name`: Uses dss to store data, set to the dss storage volume group name, such as `+vg1`

- `directory name`: Local directory storage

**Default Value**: `-`
