# GUC Parameters

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:02:33.954Z pushedAt=2026-08-19T10:31:46.949Z -->

## **_INDEX_BUFFER_SIZE**

**Parameter Description**  
A global cache used to store read-only copies of index hot pages (currently mainly B-tree index root node pages). This cache is intended to reduce contention for high-concurrency access to index pages, thereby improving system concurrency performance.

**Value Range**

- Minimum value: `16K` (16,384 bytes)

- Maximum value: `32T` (35,184,372,088,832 bytes)

- Unit: byte

**Default Value**  
`8M` (8,388,608 bytes)

## **_OPT_CBO_STAT_SAMPLING_LEVEL**

**Parameter Description**  
Sets the dynamic sampling level, which is used by the optimizer to collect statistics information. This parameter determines whether the optimizer adopts dynamic sampling and the sampling ratio when generating an execution plan in the absence of statistics information or when the statistics information is outdated.

**Value Range**

- Integer

- Minimum value: `0`, indicating that the dynamic sampling feature is disabled.

- Maximum value: `9`

**Default Value**  
`0`

## **_PREFETCH_ROWS**

**Parameter Description**  
Sets the number of rows that the system prefetches into the cache during data retrieval.

**Value Range**

- Integer

- Minimum value: `1`

- Maximum value: `2^32 - 1`

**Default Value**  
`100`

## **CHECKPOINT_PERIOD**

**Parameter Description**  
Sets the maximum period between two checkpoints executed by the system. When this preset period is reached, a checkpoint is automatically triggered.

**Feature**

- **Alias**: `CHECKPOINT_TIMEOUT` (modification is not recommended).

**Value Range**

- Integer, unit: second

- Minimum value: `1` 

- Maximum value: `2^32 - 1` 

**Default Value**  
`300` 

## **CHECKPOINT_PAGES**

**Parameter Description**  
Sets the threshold for the number of dirty pages that triggers a checkpoint. When the number of dirty pages in the database buffer reaches this value, a checkpoint is automatically triggered.

**Feature**

- **Alias**: `CHECKPOINT_INTERVAL` (modification is not recommended).

**Value Range**

- Integer, unit: page

- Minimum value: `1` 

- Maximum value: `2^32 - 1` 

**Default Value**  
`100,000` 

## **REACTOR_THREADS**

**Parameter Description**  
Sets the number of threads used for I/O listening. These threads are responsible for receiving and distributing connection requests and network packets from clients.

**Configuration Recommendation**

- **Recommended value**: `OPTIMIZED_WORKER_THREADS / 50`.

- **Risk warning**:

  1. Setting an excessively large value consumes additional CPU, memory, and thread resources. When system resources are insufficient, it may cause abnormal database operation.

  2. The value must not exceed the total number of Agent threads.

**Value Range**  
[1, 10000]

**Default Value**  
`1`

## **OPTIMIZED_WORKER_THREADS**

**Parameter Description**  
Sets the number of optimal worker threads for the database. This parameter determines whether the system adopts **binding mode** or **separation mode** between sessions and threads.

**Working Mode**

- When the number of sessions > `OPTIMIZED_WORKER_THREADS`, the system switches to the session-thread separation mode; otherwise, it uses the binding mode.

**Relationship with MAX_WORKER_THREADS**

1. **At startup**: If `OPTIMIZED_WORKER_THREADS > MAX_WORKER_THREADS` in the configuration file, `MAX_WORKER_THREADS` is automatically adjusted to the value of this parameter.

2. **After startup**:

    - It can be set online to any value less than or equal to the current `MAX_WORKER_THREADS`.

    - If an attempt is made to set it to a value greater than the current `MAX_WORKER_THREADS`, the system reports an error.

3. **After restart**: `MAX_WORKER_THREADS` retains its original value and is not changed by the modification of this parameter.

**Configuration Recommendation and Constraints**

- **Upper limit for the maximum value**: It is not recommended to exceed the value of the `SESSIONS` parameter; otherwise, thread resources will be left idle and wasted.

- **Resource consumption**:

  - Each worker thread occupies approximately **0.5 MB+** of memory.

  - Setting it too large will significantly increase CPU and memory overhead, and may cause database exceptions when resources are insufficient.

- **Absolute upper limit**: Because security functions are restricted to a maximum input memory of 2 GB, this parameter cannot exceed **10000**.

**Value Range**  
[2, 10000]

**Default Value**  
`100`

## **MAX_WORKER_THREADS**

**Parameter Description**  
Sets the maximum capacity of the database worker thread pool.

**Relationship with OPTIMIZED_WORKER_THREADS**

1. **Numeric relationship**: The value of this parameter **must be greater than or equal to** `OPTIMIZED_WORKER_THREADS`. If an attempt is made to set it to a value smaller than the current `OPTIMIZED_WORKER_THREADS`, the system reports an error.

2. **Thread expansion**: When the number of active sessions exceeds `OPTIMIZED_WORKER_THREADS`, the number of worker threads can continue to expand from the optimal value until it reaches the **maximum value** set by this parameter. That is, the number of additionally expandable threads is `MAX_WORKER_THREADS - OPTIMIZED_WORKER_THREADS`.

**Configuration Recommendation and Constraints**

- **Reference for the maximum value**: `OPTIMIZED_WORKER_THREADS` is not recommended to exceed the value of the `SESSIONS` parameter; otherwise, thread resources will be wasted.

- **Resource consumption**:

  - Each worker thread occupies approximately **0.5 MB+** of memory.

  - Setting it too large will significantly increase CPU and thread management overhead, and may cause database exceptions when resources are insufficient.

- **Absolute upper limit**: Limited by the memory of system security functions, the maximum value of this parameter is **10000**.

**Value Range**  
[2, 10000]

**Default Value**  
`100`

## **STATS_LEVEL**

**Parameter Description**  
Controls whether to collect and count the Data Manipulation Language (DML) operation information performed on tables.

**Feature**

- **Alias**: `STATISTICS_LEVEL` (modification is not recommended).

**Value Range and Behavior**

| Value  | Meaning and Behavior |
| --- | --- |
| **TYPICAL** or **ALL** | Enables table-monitoring.<br>• The system collects statistics about DML operations performed on tables.<br>• The collected statistics are displayed in the system table `SYS_DML_STATS` after about 15 minutes. |
| **BASIC** | Disables table-monitoring.<br>• The above DML operation statistics are not collected. |

**Default Value**  
`TYPICAL`

## use\_bison\_parser<a name="use_bison_parser"></a>

**Parameter Description**  
Specifies whether to use the Bison parser to parse common SQL statements.

This parameter takes effect immediately after being set. For details about how to set it, see the corresponding setting method in [Configuring Database System Parameters](../database_administration_guide/basic_management_of_database_system/configuring_the_database_system.md#configuring-database-system-parameters).

**Value range**: Boolean

- `on`/`true` indicates that the Bison parser is used.

- `off`/`false` indicates that the native parser is used.

**Default value**: `on`

>[!NOTE]
>
>In common SQL statements, floating-point literals ending with f/d are controlled by this parameter.

## **BUF_POOL_NUM**

**Parameter Description**  
Sets the number of partitions for the data buffer.

**Value Range**  
[0, 1024]

**Default Value**  
`0`

**NOTE**

- When this parameter is **not configured** or **explicitly set to `0`**, the system **automatically calculates and adjusts** a reasonable number of partitions based on the actual value of `DATA_BUFFER_SIZE`.

## BUFFER_PAGE_CLEAN_PERIOD

**Parameter Description**  
Controls the execution interval of the Page Clean scheduled task (unit: second).
Page Clean is a mechanism for quickly cleaning dirty pages in the buffer. In regular business scenarios, this scheduled task usually does not need to be enabled, because dirty page cleaning can be automatically triggered by the buffer eviction mechanism. However, in sustained high-load performance test scenarios such as TPC-C and sysbench, if the disk I/O capability is strong, this scheduled task can be enabled to accelerate dirty page flushing to disk.

**Value range**: integer, [0, 2^32 - 1]  
**Default value**: `0` (Scheduled cleaning disabled)

## INTERCONNECT_CHANNEL_NUM

**Parameter Description**  
Configures the number of internal communication (MES) connection channels.

**Value range**: Integer, [1, 32]  
**Default value**: `4`

## LOG_BUFFER_COUNT

**Parameter Description**  
Sets the number of log buffers.

**Value range**: Integer, (0, 16]  
**Default value**: `4`

## LOG_BUFFER_SIZE

**Parameter Description**  
Defines the size of each log buffer, which is used to cache REDO logs.
Properly increasing this value helps improve data write performance. Adjust it based on the actual available memory.

**Value range**: Integer, [1M, 128M] (Unit: byte)  
**Default value**: `4M`

## TIMED_STATS (Alias: TIMED_STATISTICS, not recommended)

**Parameter Description**  
Controls whether to collect time-related statistics.

- **TRUE**: Collects and stores time-related statistics (queryable through the `DV_SYS_STATS` dynamic view or trace files)

- **FALSE**: Sets all time-related statistics values to 0

**Default value**: `TRUE`

## SQL_STAT

**Parameter Description**  
Specifies Whether to enable SQL performance statistics views. The setting applies to the entire database instance.

- **TRUE**: Enables SQL performance statistics.

- **FALSE**: Disables SQL performance statistics.

**Default value**: `TRUE`

## MES_ELAPSED_SWITCH

**Parameter Description**  
Specifies whether to enable the latency statistics function for MES internal communication.

- **TRUE**: Enables latency statistics.

- **FALSE**: Disables latency statistics.

**Default value**: `FALSE`

## SESSIONS

**Parameter Description**  
Sets the maximum number of concurrent sessions in a database instance. The number of concurrent sessions at runtime must not exceed this value.

### Session Composition

- **System-reserved sessions**:

  - Fixed 32 sessions: used for internal tasks (such as resource reclamation, checkpoint, etc.)

  - `PARALLEL_MAX_THREADS` sessions: used for the SQL parallel execution framework

- **EMERG user sessions**: the quantity is controlled by the `SUPER_USER_RESERVED_SESSIONS` parameter

- **Regular user sessions**:

  - At least 1 must be configured; otherwise, the database cannot start.

  - **When parallel replay is disabled**:

    Number of regular user sessions = `SESSIONS − 32 − PARALLEL_MAX_THREADS − SUPER_USER_RESERVED_SESSIONS`

  - **When parallel replay is enabled (standby only)**:

    Available sessions = `SESSIONS − 32 − PARALLEL_MAX_THREADS − SUPER_USER_RESERVED_SESSIONS − LOG_REPLAY_PROCESSES − REPLAY_PRELOAD_PROCESSES`  
    (The behavior on the primary is the same as when parallel replay is disabled.)

### Configuration Recommendation

- Generally, it is recommended to set it to `OPTIMIZED_WORKER_THREADS × 1.2`

- On DN nodes, `SESSIONS < OPTIMIZED_WORKER_THREADS` is recommended.

### Effective Mode

- **Take effect immediately**: The new value ≥ the initial value `init_count` and ≤ `1.5 × init_count`

- **Take effect after a restart**: The new value < `init_count` or > `1.5 × init_count` (but ≤ 16320)

  > `init_count` is the `SESSIONS` value at database startup. The system reserves 50% of the expansion space by default for dynamic adjustment, so real-time adjustment supports only expansion, not reduction.

### Precautions

1. After `SUPER_USER_RESERVED_SESSIONS` is modified, a restart is required for the change to take effect.

2. Before adjusting `SESSIONS`, ensure that `_MAX_RM_COUNT` satisfies the constraint conditions.

3. If `SESSIONS > 13600`, `_MAX_RM_COUNT` must be a multiple of 64 and greater than `SESSIONS`. It is recommended to set it to 16320.

**Value range**: integer
Lower limit = `33 + PARALLEL_MAX_THREADS + SUPER_USER_RESERVED_SESSIONS`
Upper limit = `16320`  
**Default value**: `200`

## **USE_RBP**

**Parameter Description**  
Master switch for RBP. After it is enabled, the database can work with RBPS to save and read the pages required for recovery, shortening the recovery time after an instance or node failure.

Disabled by default: When RBPS is not deployed or accelerated recovery is not required, disabling this capability avoids introducing additional resource overhead.

**Value Range**

- `TRUE`: Enables RBP.

- `FALSE`: Disables RBP.

**Default Value**  
`FALSE`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- When RBPS is not deployed or only ordinary recovery is performed, keep the default value.

- When RBPS is required for accelerated recovery, set this parameter to `TRUE` for all related database nodes, and confirm that the RBPS service has been deployed and the network is reachable.

## **RBP_FOR_RECOVERY**

**Parameter Description**  
Controls whether RBP acceleration is allowed during the recovery phase. This parameter must be used together with `USE_RBP=TRUE`; when RBPS is available and the recovery conditions are met, the database can use pages in RBPS to accelerate the recovery.

Enabled by default: After you explicitly enable RBP, it is expected that the recovery phase directly uses the acceleration capability; this independent switch is also retained to facilitate temporarily disabling accelerated recovery.

**Value Range**

- `TRUE`: RBP is allowed during the recovery phase.

- `FALSE`: RBP is not used during the recovery phase.

**Default Value**  
`TRUE`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- Keep the default value for regular accelerated recovery scenarios.

- When RBP accelerated recovery needs to be temporarily disabled, set it to `FALSE` and restart the instance.

## **RBP_IP**

**Parameter Description**  
Configures the target address list for the database to connect to RBPS.

In a two-node DTC deployment, the database side is usually configured to connect to the peer RBPS. It is recommended that RBP communication use a different NIC from service access.

**Value Range**

- A non-empty string.

- A single address or a comma-separated list of addresses can be configured.

**Default Value**  
`127.0.0.1`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- In a two-node DTC scenario, the peer RBPS address is usually configured, for example, [ip1, ip2].

- If RBP uses a dedicated NIC, it is recommended to configure the address on the RBP network instead of the service access address.

## **RBP_PORT**

**Parameter Description**  
Configures the TCP port for connecting the database to RBPS. This port must be consistent with the listening port configured in the RBPS server `rbps.conf`.

**Value Range**  
Integer, `[1024, 65535]`

**Default Value**  
`2611`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- If the port needs to be changed, the `RBP_PORT` of all database nodes and the listening ports of all related RBPS servers must be modified synchronously, and the corresponding components need to be restarted.

## **LOCAL_RBP_HOST**

**Parameter Description**  
Configures the local address that the current database instance reports to RBPS. This parameter is used to identify the source database node in RBPS-side logs, status records, and read/write phase notifications. It does not determine which RBPS the database connects to; the connection target is determined by `RBP_IP`.

The default value is the local loopback address.

**Value Range**  
A non-empty string.

**Default Value**  
`127.0.0.1`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- In a cluster deployment, configure it as the RBP communication IP or stable hostname that can be recognized and distinguished by RBPS for the current database node.

- If RBP uses an independent NIC, it is recommended to configure the address corresponding to the independent NIC.

## **RBP_RT_ANALYSIS**

**Parameter Description**  
Controls whether to enable the DTC RBP runtime analysis capability. After it is enabled, the database prepares part of the information required for recovery in advance during runtime, thereby reducing the recovery preparation time after a fault occurs.

Disabled by default: Runtime analysis introduces additional resource overhead and should be enabled on demand in scenarios where reducing recovery time is explicitly required.

**Value Range**

- `TRUE`: Enables runtime analysis.

- `FALSE`: Disables runtime analysis.

**Default Value**  
`FALSE`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- This parameter is meaningful only when `USE_RBP=TRUE` and `RBP_FOR_RECOVERY=TRUE`.

- When sensitive to runtime resource consumption, or when not in a DTC accelerated recovery scenario, keep the default value.

## **RBP_RT_PARSE_WORKERS**

**Parameter Description**  
Configures the number of parsing worker threads for DTC RBP runtime analysis.

Defaults to `2`: In a two-node scenario, two parsing worker threads can balance analysis efficiency and CPU usage, avoiding excessive background overhead during initial deployment.

**Value Range**  
Integer, `[1, 8]`

**Default Value**  
`2`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- If the runtime analysis processing speed is insufficient and CPU resources are sufficient, the value can be increased appropriately.

- If CPU pressure is high or analysis pressure is low, you can retain the default value or reduce the value.

## **RBP_RT_PAGE_OWNER_WORKERS**

**Parameter Description**  
Configures the number of worker threads for page ownership processing in DTC RBP runtime analysis.

Defaults to `4`: Page ownership processing usually requires a higher degree of concurrency than the parsing phase, and appropriately increasing the number of worker threads can reduce the risk of recovery information processing backlog.

**Value Range**  
Integer, `[1, 8]`

**Default Value**  
`4`

**Effective Mode**  
Takes effect after a restart.

**Configuration Recommendation**

- Keep the default value in normal cases.

- If the runtime analysis backlog is significant and memory and CPU resources are sufficient, increase the value appropriately.

## **RBP_ASSEMBLE_MAX_SCAN**

**Parameter Description**  
Configures the maximum number of candidates to be scanned in a single pass when RBP writes pages. Increasing this value appropriately can improve batch page sending efficiency, but may also increase scanning overhead.

Defaults to `300`: This value strikes a balance between batch sending efficiency and runtime overhead, and is suitable for most scenarios.

**Value Range**  
Integer, `[100, 1000000]`

**Default Value**  
`300`

**Effective Mode**  
Takes effect immediately.

**Configuration Recommendation**

- Keep the default value in normal cases.

## Database Parameter Configuration Description

### 1. Global Shared Memory

#### `TEMP_BUFFER_SIZE`

**Function**: Configures the total size of the temporary buffer, which is used to support temporary operations such as materialized table creation and data sorting.

- At system startup, the buffer is evenly divided into multiple temporary pools based on `TEMP_BUFFER_SIZE` (total size) and `TEMP_POOL_NUM` (number of temporary pools).

- The minimum size of a single temporary pool is **2 GB**. If `TEMP_BUFFER_SIZE < 2 GB`, only one temporary pool is created.

- Supports **online dynamic expansion**:

  - The space expanded each time must be greater than or equal to `temp_buffer_extend_threshold`.

  - Number of newly added temporary pools = `MIN(expanded value / size of a single temporary pool, 1)`

  - The maximum total number of temporary pools is **128**.

- **Online reduction is not supported**: To reduce this parameter, modify the database configuration file or use `SCOPE=PFILE`, and **restart the database** for the change to take effect.

**Value range**: integer, [32M, 4T] (Unit: byte)  
**Default value**: 32M  
**Installation script default value**: When `install.py` is used for installation, it is automatically set to **1G**.

#### `DATA_BUFFER_SIZE`

**Function**: Sets the size of the data buffer, which is used to cache recently accessed data pages and improve data access efficiency.

- It should be configured reasonably based on the physical memory of the server.

- **Takes effect dynamically**, but is subject to the following parameter constraints:

  - Minimum value = `BUFFER_UINT_SIZE`

  - Maximum value = `BUFFER_UINT_SIZE × BUF_POOL_NUM × 1024`

- If `BUFFER_CHECK_MEMORY = TRUE`, the configured value **must not exceed the total physical memory**, otherwise an error is triggered in the following scenarios:

  - Parameter configuration phase

  - Online expansion operation

  - Loading configuration at database startup

**Value range**: integer, [64M, 32T], supported units: B / K / M / G / T (Byte by default when no unit is specified)  
**Default value**: 128M  
**Configuration recommendation**:

  - It is recommended to set `DATA_BUFFER_SIZE` to an integer multiple of `BUFFER_UINT_SIZE × BUF_POOL_NUM` to reduce memory fragmentation.

  - **Only online expansion is supported**; to decrease it, modify the configuration file or use `SCOPE=PFILE`, and **restart the database** for it to take effect.

**Installation script default value**: When `install.py` is used for installation, it is automatically set to **2G**.

#### `SHARED_POOL_SIZE`

**Function**: Configures the total size of the shared pool, including shared memory structures such as the Lock Pool, SQL Pool, and DC Pool.

- **The minimum value is affected by `init_lock_pool_pages`**:

  - Only when `init_lock_pool_pages = 128` is configured, `SHARED_POOL_SIZE` is allowed to be set to the minimum value of **68M**.

- The `SHARED_POOL_SIZE` of the standby node **must be greater than or equal to the primary node value**; otherwise, the **68M** minimum configuration cannot be used.

- After increasing this parameter, it is recommended to evaluate whether `init_lock_pool_pages` needs to be increased accordingly.

- **Dynamic adjustment restrictions**:

  - Only **online expansion** is supported; online reduction is not supported.

  - The single expansion amount (`expand_size`) must be greater than or equal to `MIN(512M, current SHARED_POOL_SIZE)`; otherwise, it takes effect only after a restart.

  - A maximum of **15** dynamic expansions is supported; beyond this limit, subsequent modifications take effect only after the database is restarted.

**Value range**: integer, [68M, 32T] (Unit: byte)  
**Default value**: 128M  
**Configuration recommendation**:

  - To reduce the value, you must modify the configuration file or use `SCOPE=PFILE`, and **restart the database** for the change to take effect.  

**Installation script default value**: When `install.py` is used for installation, it is automatically set to **1G**.

### 2. Checkpoint

#### `CHECKPOINT_IO_CAPACITY`

**Function**: Controls the number of dirty pages (in pages) written to the disk each time a checkpoint is triggered.

**Value range**: Integer, [1, 4096]  
**Default value**: 1024

#### `_CHECKPOINT_TIMED_TASK_DELAY`

**Function**: Sets the interval between two scheduled checkpoint tasks (in milliseconds).

- Scheduled checkpoint is jointly controlled by `checkpoint_pages` and `checkpoint_period`. This parameter is used to adjust the execution frequency.

**Value range**: integer, [0, 2^32 - 1] (unit: millisecond)  
**Default value**: 100

### 3. Transaction

#### `_UNDO_AUTO_SHRINK`

**Function**: Controls whether to enable the automatic shrink feature of Undo Segment.

**Value range**:

- `true`: Enables automatic shrink.

- `false`: Disables automatic shrink.

**Default value**: `true`

### 4. Process

#### `DBWR_PROCESSES`

**Function**: Sets the number of background dirty page writer (DBWR) threads.

- Increasing this value improves I/O concurrency, but increases CPU and memory overhead.

- **The default configuration varies by installation package type**:

  - Shared storage installation package: defaults to **36**

  - Other types of installation packages: defaults to **8**

- **Special recommendation**: When `FILE_OPTIONS = ASYNCH` or `SETALL` (that is, asynchronous I/O is enabled), it is recommended to set this parameter to **1**, because asynchronous I/O already provides high concurrency capability and does not require additional DBWR threads.

**Value range**: integer, [1, 36]  
**Default value**: 1

### 5. Session-level Parameters

#### `_DOUBLEWRITE`

**Function**: Whether to enable the double write mechanism, which improves the reliability of data writes and prevents partial write issues.

**Value range**:

- `TRUE`: Enables double write.

- `FALSE`: Disables double write.

**Default value**: `TRUE`

> Note: For all parameters that only support online expansion, if a reduction is required, it must be modified through the configuration file or `SCOPE=PFILE`, and takes effect only after **restarting the database instance**.
