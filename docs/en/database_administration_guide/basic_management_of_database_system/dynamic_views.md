# Dynamic Views

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:00:13.976Z pushedAt=2026-08-20T03:31:28.799Z -->

## DV_DSS_TIME_STATS

DSS device information view. It supports collecting operation statistics of DSS devices in the database, recording the wait time and execution count of each DSS operation. It can diagnose I/O bottlenecks and latency issues at the storage layer, helping DBAs identify storage performance problems such as high-latency read and write operations.

Field description:

| No. | Field Name | Data Type | Description |
| --- | ---------------------------- | ------------- | ---------------- |
| 0   | PREAD_WAIT_COUNT             | BINARY_BIGINT | Physical read wait count |
| 1   | PREAD_WAIT_TIME              | BINARY_BIGINT | Physical read wait time |
| 2   | PWRITE_WAIT_COUNT            | BINARY_BIGINT | Physical write wait count |
| 3   | PWRITE_WAIT_TIME             | BINARY_BIGINT | Physical write wait time |
| 4   | PREAD_SYN_META_WAIT_COUNT    | BINARY_BIGINT | Synchronous metadata physical read wait count |
| 5   | PREAD_SYN_META_WAIT_TIME     | BINARY_BIGINT | Synchronous metadata physical read wait time |
| 6   | PWRITE_SYN_META_WAIT_COUNT   | BINARY_BIGINT | Synchronous metadata physical write wait count |
| 7   | PWRITE_SYN_META_WAIT_TIME    | BINARY_BIGINT | Synchronous metadata physical write wait time |
| 8   | PREAD_DISK_WAIT_COUNT        | BINARY_BIGINT | Physical disk read wait count |
| 9   | PREAD_DISK_WAIT_TIME         | BINARY_BIGINT | Physical disk read wait time |
| 10  | PWRITE_DISK_WAIT_COUNT       | BINARY_BIGINT | Physical disk write wait count |
| 11  | PWRITE_DISK_WAIT_TIME        | BINARY_BIGINT | Physical disk write wait time |
| 12  | FOPEN_WAIT_COUNT             | BINARY_BIGINT | Wait count of file open operations |
| 13  | FOPEN_WAIT_TIME              | BINARY_BIGINT | Wait time of file open operations |
| 14  | STAT_WAIT_COUNT              | BINARY_BIGINT | Wait count of obtaining file status |
| 15  | STAT_WAIT_TIME               | BINARY_BIGINT | Wait time of obtaining file status |
| 16  | FIND_FT_ON_SERVER_WAIT_COUNT | BINARY_BIGINT | Wait count of searching for file tokens on the server |
| 17  | FIND_FT_ON_SERVER_WAIT_TIME  | BINARY_BIGINT | Wait time of searching for file tokens on the server |
| 18  | LOCK_VG_WAIT_COUNT           | BINARY_BIGINT | Wait count of locking volume groups |
| 19  | LOCK_VG_WAIT_TIME            | BINARY_BIGINT | Wait time of locking volume groups |
| 20  | LATCH_CONTEXT_WAIT_COUNT     | BINARY_BIGINT | Latch context wait count |
| 21  | LATCH_CONTEXT_WAIT_TIME      | BIGINT        | Latch context wait time |

## DV_SQL_EXECUTION

A view of historical execution plans for SQL statements in the database. You can query the execution plan and status information of each SQL statement historically executed in the database for SQL performance analysis and optimization, and compare the performance differences of different execution plans through historical data.

Field description:

| No. | Field Name | Data Type | Description |
| --- | ---------------------- | ------------- | ----------------------------------- |
| 0   | SQL_ID                 | VARCHAR       | Unique identifier of the SQL statement |
| 1   | PLAN_VERSION           | VARCHAR       | Version number of the execution plan |
| 2   | EXECUTION_COUNT        | BINARY_BIGINT | Execution count |
| 3   | DISK_READ_COUNT        | BINARY_BIGINT | Disk read count |
| 4   | BUFFER_GET_COUNT       | BINARY_BIGINT | Buffer get count |
| 5   | CR_GET_COUNT           | BINARY_BIGINT | Consistent read count (the number of times the SQL statement searches in the CR pool) |
| 6   | SORT_COUNT             | BINARY_BIGINT | Sort count |
| 7   | HARD_PARSE_TIME        | BINARY_BIGINT | Hard parse time (microseconds) |
| 8   | SOFT_PARSE_TIME        | BINARY_BIGINT | Soft parse time (microseconds) |
| 9   | IO_WAIT_ELAPSED_TIME   | BINARY_BIGINT | I/O wait time (microseconds) |
| 10  | CON_WAIT_ELAPSED_TIME  | BINARY_BIGINT | Condition wait time (microseconds) |
| 11  | CPU_ELAPSED_TIME       | BINARY_BIGINT | CPU execution time (microseconds) |
| 12  | TOTAL_ELAPSED_TIME     | BINARY_BIGINT | Total execution time (microseconds) |
| 13  | LAST_LOAD_TIMESTAMP    | DATE          | Last load timestamp (the time of the most recent load into the library cache, generally the time of the first hard parse) |
| 14  | LAST_ACTIVE_TIMESTAMP  | DATE          | Last active timestamp (generally the last execution time) |
| 15  | REFERENCE_COUNT        | BINARY_UINT32 | SQL reference count (the number of sessions or processes currently using this execution plan) |
| 16  | MEMORY_PAGES           | BINARY_UINT32 | Number of pages occupied by the context |
| 17  | SHARED_MEMORY_SIZE     | BINARY_BIGINT | Size of occupied memory |
| 18  | VM_OPEN_PAGE_COUNT     | BINARY_BIGINT | Number of opened VM pages |
| 19  | VM_CLOSE_PAGE_COUNT    | BINARY_BIGINT | Number of closed VM pages |
| 20  | VM_SWAPIN_PAGE_COUNT   | BINARY_BIGINT | Number of VM pages swapped in from disk to memory |
| 21  | VM_FREE_PAGE_COUNT     | BINARY_BIGINT | Number of freed VM pages (used for execution) |
| 22  | PLAN_TEXT              | VARCHAR       | Execution plan text |
| 23  | VM_MAX_OPEN_PAGE_COUNT | BINARY_BIGINT | Maximum number of VM pages opened by the SQL execution |
| 24  | VM_SWAPOUT_PAGE_COUNT  | BINARY_BIGINT | Number of VM pages swapped out from memory to disk |
| 25  | VM_ALLOC_PAGE_COUNT    | BINARY_BIGINT | Number of allocated VM pages |
| 26  | SIGNATURE              | VARCHAR       | Execution plan signature |
| 27  | EXPLAIN_ID             | VARCHAR       | Execution plan ID |

## DV_SLOW_SQL

Slow SQL view. It records slow SQL statements whose execution time exceeds the preset threshold, including key information such as SQL text, execution time, wait events, and lock waits, which can be used for performance optimization.

The slow SQL view information is obtained from slow SQL logs. Therefore, to obtain slow SQL information, you need to enable slow SQL logging:

```
--Enable slow SQL logging, corresponding to _LOG_LEVEL = 256
alter system set SLOWSQL_LOG_MODE=ON;
--Set the threshold for slow SQL logging, used to determine whether a SQL statement should be logged as slow
alter system set SQL_STAGE_THRESHOLD=2;
--Check the setting result
show parameter SLOWSQL_LOG_MODE;
show parameter SQL_STAGE_THRESHOLD;
```

Field description:

| No. | Field Name                    | Data Type          | Description                           |
| --- | ---------------------- | ------------- | ---------------------------- |
| 0   | TENANT_ID              | BINARY_UINT32 | Tenant ID                         |
| 1   | CTIME                  | VARCHAR       | Execution time                         |
| 2   | STAGE                  | VARCHAR       | SQL execution stage                      |
| 3   | SID                    | BINARY_BIGINT | Session ID                         |
| 4   | CLIENT_IP              | VARCHAR       | Client IP address                      |
| 5   | ELAPSED_TIME           | NUMBER        | Total execution time (microseconds)                    |
| 6   | PARAMS                 | VARCHAR       | SQL statement bind parameters                    |
| 7   | SQL_ID                 | VARCHAR       | SQL statement ID                      |
| 8   | EXPLAIN_ID             | VARCHAR       | Execution plan ID                       |
| 9   | DISK_READ_COUNT        | BINARY_BIGINT | Disk read count                       |
| 10  | BUFFER_GET_COUNT       | BINARY_BIGINT | Buffer get count                      |
| 11  | CR_GET_COUNT           | BINARY_BIGINT | Consistent read count (the number of times the SQL statement searches in the CR pool) |
| 12  | DIRTY_COUNT            | BINARY_BIGINT | Number of generated dirty pages |
| 13  | PROCESSED_ROWS         | BINARY_BIGINT | Number of processed rows |
| 14  | CPU_ELAPSED_TIME       | BINARY_BIGINT | CPU execution time (microseconds)                  |
| 15  | IO_WAIT_ELAPSED_TIME   | BINARY_BIGINT | I/O wait time (microseconds)                   |
| 16  | CON_WAIT_ELAPSED_TIME  | BINARY_BIGINT | Condition wait time (microseconds)                   |
| 17  | PARSE_ELAPSED_TIME     | BINARY_BIGINT | Reparse time during execution (microseconds)                |
| 18  | VM_ALLOC_PAGES         | BINARY_BIGINT | Number of allocated VM pages                       |
| 19  | VM_MAX_OPEN_PAGE_COUNT | BINARY_BIGINT | Maximum number of VM pages opened                 |
| 20  | TOP1_EVENT             | VARCHAR       | Longest execution wait event                    |
| 21  | TOP1_WAIT_TIME         | BINARY_BIGINT | Wait time of the longest execution wait event                |
| 22  | TOP1_EVENT_COUNT       | BINARY_BIGINT | Occurrence count of the longest execution wait event                |
| 23  | TOP2_EVENT             | VARCHAR       | Second longest execution wait event                   |
| 24  | TOP2_WAIT_TIME         | BINARY_BIGINT | Wait time of the second longest execution wait event               |
| 25  | TOP2_EVENT_COUNT       | BINARY_BIGINT | Occurrence count of the second longest execution wait event               |
| 26  | TOP3_EVENT             | VARCHAR       | Third longest execution wait event                   |
| 27  | TOP3_WAIT_TIME         | BINARY_BIGINT | Wait time of the third longest execution wait event               |
| 28  | TOP3_EVENT_COUNT       | BINARY_BIGINT | Occurrence count of the third longest execution wait event               |
| 29  | SQL_TEXT               | VARCHAR       | SQL statement text                      |
| 30  | EXPLAIN_TEXT           | VARCHAR       | Execution plan text                       |

**NOTE**

1. Only statements whose execution time exceeds the preset threshold and that are DML statements (SELECT, UPDATE, INSERT, DELETE, MERGE, REPLACE) are recorded.

2. STAGE indicates the SQL execution stage (PREPARE / EXECUTE / PREP_EXEC / QUERY / FETCH). When a command in a certain stage of an SQL statement times out, it is recorded in the slow SQL log. When the execution time of an SQL statement times out, it is also recorded in the slow SQL log, and the recorded STAGE is EXECUTE.

## DV_DRC_BUF_INFO

Local node buffer page view. It can monitor the buffer page usage of the local database node, display page status information, and can be used for memory management analysis, especially when lock conflicts or page conversion delays occur.

Field description:

| No. | Field Name | Data Type | Description |
| --- | ------------------- | ------------- | ---------------------------------------------------- |
| 0 | IDX | BINARY_UINT32 | Buffer resource index |
| 1 | FILE_ID | BINARY_UINT32 | File ID |
| 2 | PAGE_ID | BINARY_UINT32 | Page ID |
| 3 | CLAIMED_OWNER | BINARY_UINT32 | Node ID held by the owner |
| 4 | LOCK | BINARY_UINT32 | Lock mode of the node held by the owner:<br/>0: NULL<br/>1: SHARE<br/>2: EXCLUSIVE |
| 5 | CONVERTING_INST | BINARY_UINT32 | ID of the converting node |
| 6 | CONVERTING_CUR_LOCK | VARCHAR | Current lock mode of the converting node:<br/>NULL<br/>SHARE<br/>EXCLUSIVE |
| 7 | CONVERTING_REQ_LOCK | VARCHAR | Lock mode requested by the converting node:<br/>NULL<br/>SHARE<br/>EXCLUSIVE |
| 8 | CONVERTQ_LEN | BINARY_UINT32 | Length of the request queue of the converting node |
| 9 | EDP_MAP | BINARY_BIGINT | Bitmap of nodes holding the EDP (invalid values are set to 255) |
| 10 | CONVERTING_SID | BINARY_UINT32 | Session ID of the converting node |
| 11 | CONVERTING_RSN | BINARY_UINT32 | Serial number of the converting node |
| 12 | PART_ID | BINARY_UINT32 | ID of the associated DRC partition list |
| 13 | READONLY_COPIES | BINARY_BIGINT | Bitmap of read-only copies |
| 14 | LATEST_EDP | BINARY_UINT32 | ID of the node holding the latest EDP |
| 15 | IN_RECOVERY | BINARY_UINT32 | Whether the page is in recovery |
| 16 | REFORM_PROMOTE | BINARY_UINT32 | Whether DRC is promoted from a replica to the owner during reform |
| 17 | LSN | BINARY_BIGINT | Maximum LSN among all EDPs of this page in the cluster |
| 18 | RECOVERY_SKIP | BINARY_UINT32 | Whether DRC skips replay during reform, that is, whether dirty pages need to be persisted and flushed to disk |
| 19 | IS_RECYCLING | BINARY_UINT32 | Whether it is being recycled |
| 20 | MASTER_ID | BINARY_UINT32 | Master node ID |

## DV_DRC_LOCAL_LOCK_INFO

Lock information view of the local database node. It can be used to monitor lock usage on the local database node, helping identify issues such as lock contention and deadlocks.

Field description:

| No.  | Field Name                 | Data Type          | Description                                                    |
| --- | ------------------- | ------------- | ----------------------------------------------------- |
| 0   | IDX                 | BINARY_UINT32 | Index of the local lock resource in the resource pool                                          |
| 1   | DRID_TYPE           | VARCHAR       | Resource ID type                                                |
| 2   | DRID_UID            | BINARY_UINT32 | Resource UID                                                 |
| 3   | DRID_ID             | BINARY_UINT32 | Lock ID                                                   |
| 4   | DRID_IDX            | BINARY_UINT32 | Resource index ID                                                |
| 5   | DRID_PART           | BINARY_UINT32 | Resource partition                                                  |
| 6   | DRID_PARENTPART     | BINARY_UINT32 | Resource parent partition                                                 |
| 7   | IS_OWNER            | UINT32        | Whether it is the owner                                                |
| 8   | IS_LOCKED           | UINT32        | Whether it is locked                                                 |
| 9   | COUNT               | BINARY_UINT32 | Counter (table locks only)                                              |
| 10  | LATCH_SHARE_COUNT   | UINT32        | Latch share count                                                |
| 11  | LATCH_STAT          | UINT32        | Latch status                                                  |
| 12  | LATCH_SID           | UINT32        | Session ID holding the latch                                             |
| 13  | IS_RELEASING        | UINT32        | Whether the local lock is released                                           |
| 14  | LOCK_MODE           | UINT32        | Local lock mode                                               |

## DV_BUF_CTRL_INFO

View of buffer control block information for the local database node. It can be used for buffer management and page status analysis.

Field description:

| No.  | Field Name               | Data Type          | Description                                                                                  |
| --- | ----------------- | ------------- | ----------------------------------------------------------------------------------- |
| 0   | POOL_ID           | BINARY_UINT32 | Buffer pool ID                                                                               |
| 1   | FILE_ID           | BINARY_UINT32 | Data file ID                                                                              |
| 2   | PAGE_ID           | BINARY_UINT32 | Page ID                                                                                |
| 3   | LATCH_SHARE_COUNT | BINARY_UINT32 | Latch share count                                                                              |
| 4   | LATCH_STAT        | BINARY_UINT32 | Latch status                                                                                |
| 5   | LATCH_SID         | BINARY_UINT32 | Session ID holding the latch                                                                           |
| 6   | LATCH_XSID        | BINARY_UINT32 | Session ID of the last exclusive latch holder                                                                     |
| 7   | IS_READONLY       | BINARY_UINT32 | Whether the page is read-only                                                                                |
| 8   | IS_DIRTY          | BINARY_UINT32 | Whether the page is dirty                                                                                |
| 9   | IS_REMOTE_DIRTY   | BINARY_UINT32 | Whether the page is remotely dirty                                                                              |
| 10  | IS_MARKED         | BINARY_UINT32 | Whether the page is marked                                                                                |
| 11  | LOAD_STATUS       | BINARY_UINT32 | Load status:<br/>0: BUF_NEED_LOAD<br/>1: BUF_IS_LOADED<br/>2: BUF_LOAD_FAILED |
| 12  | IN_OLD            | BINARY_UINT32 | Whether the page is in the old list (LRU cold end)                                                                       |
| 13  | IN_CKPT           | BINARY_UINT32 | Whether the page is in the checkpoint queue                                                                            |
| 14  | LOCK_MODE         | BINARY_UINT32 | Local lock mode:<br/>0: NULL<br/>1: SHARE<br/>2: EXCLUSIVE                                       |
| 15  | IS_EDP            | BINARY_UINT32 | Whether the page is an EDP |
| 16  | EDP_SCN           | BINARY_BIGINT | Most recent SCN when the local page became an EDP                                                                    |
| 17  | EDP_MAP           | BINARY_BIGINT | Bitmap of nodes holding the EDP                                                                       |
| 18  | REF_NUM           | BINARY_UINT32 | Reference count                                                                                |
| 19  | LASTEST_LFN       | BINARY_BIGINT | Latest log file number                                                                             |
| 20  | NEED_FLUSH        | BINARY_BIGINT | Whether flushing is required during reform (fixed to 0; buf ctrl does not need flushing)                                                  |
| 21  | BEEN_LOADED       | BINARY_UINT32 | Whether the cached page has been loaded                                                                         |
| 22  | IN_RECOVERY       | BINARY_UINT32 | Whether the page is being recovered during reform                                                                    |
| 23  | LAST_CKPT_TIME    | BINARY_UINT32 | Time when the local EDP was last added to the EDP group                                                                  |
| 24  | IS_RESIDENT       | BINARY_UINT32 | Whether the page is resident in memory                                                                              |
| 25  | IS_PINNED         | BINARY_UINT32 | Whether the page is pinned                                                                             |
| 26  | PAGE_SCN          | BINARY_BIGINT | Page SCN  |
