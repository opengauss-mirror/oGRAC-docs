# DFX<a name="ZH-CN_CONCEPT_0289895656"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:46:34.431Z pushedAt=2026-08-17T00:46:29.636Z -->

## DFX Capability

Various means are provided in oGRAC to monitor the current running status of the database, assisting DBAs in problem analysis. Commonly used tools include WSR reports, statistical views, and slow SQL diagnosis.

### WSR Report

The WSR report is a commonly used performance analysis tool. It generates a performance analysis report for a specific period by taking database snapshots and comparing the statistical data collected from snapshots created at two different points in time. The default snapshot interval is 30 minutes, which can be modified by users, and snapshots can also be created manually.

The WSR report aggregates the collected statistical data across different dimensions, such as database, session, buffer hit, and host, and presents them in a unified view. This allows users to intuitively observe the database running status during the period and thereby identify existing issues.

Currently, the statistical data that can be reflected in the WSR report includes:

- CPU and memory usage of the current host
- Wait events of the current database instance
- SQL performance statistics
- Buffer wait event statistics
- Latch statistics
- Slow SQL and execution plans
- SQL being executed and session information

### Statistical Views

oGRAC provides multiple views for users to view system information. By functional dimension, they can be categorized into DBA views, user views, and performance views. By combining the information from these views, the internal running status of the current database can be observed. The following table describes some common performance views.

| View  | Description  |
|---|---|
| DV_SYSTEM  | Used to query the CPU attributes and load, as well as memory-related information of the host where the current database instance resides.  |
| DV_DATABASE  | Used to query basic information of the current database, such as name, status, SCN, and RCY_POINT. |
| DV_SESSIONS  | Used to query session information within the current database.  |
| DV_GMA  | Used to query information related to buffer sizes, such as the currently configured data buffer size, shared pool size, and log buffer size.  |
| ADM_TABLESPACES  | Used to query the usage of device files such as data files and undo files in tablespaces. |
| DV_DATA_FILES  | Used to query the current status and attributes of device files such as data files and undo files in the current database.  |
| DV_LOG_FILES  | Used to query information about the current status, real-time write position, and remaining space of log files in the current database.  |
| DV_BUFFER_POOL_STATS  | Used to query detailed information of the current data buffer, including the total number of pages, number of dirty pages, and number of available pages in each partition.  |
| DV_TEMP_POOLS  | Used to query detailed information of the current temp buffer, including the total number of pages, number of available pages, and the number of pages that can be swapped out to disk in each partition. |
| DV_LOCKS  | Used to query information about all table locks and row locks in the current database.  |
| DV_LATCHS  | Used to query statistical information about page latches and heap latches in the current database.  |
| DV_SYS_STATS  | Used to query the statistical counts of all events in the current database. |
| DV_SYS_EVENTS  | Used to query statistical information about all wait events in the current database. |
| DV_PARAMETERS  | Used to query the runtime values and default values of all configuration items in the current database. |
| DV_SQLS  | Used to query the execution status of SQL DML statements. |
| DV_LONG_SQL  | Used to query information about slow SQL statements. Only SQL statements whose execution time exceeds LONGSQL_TIMEOUT can be queried. |
| DV_SESSION_EVENTS  | Used to query the events existing in the current session. |
| DV_SESSION_WAITS  | Used to query the wait events existing in the current session. |
| DV_BUFFER_ACCESS_STATS  | Used to query statistical information about buffer access in the current database. |
| DV_UNDO_STATS  | Used to query the usage information of undo space within the last 10 minutes. |
| DV_UNDO_SEGMENTS  | Used to query the real-time status information of all undo segment queues. |
| DV_ARCHIVED_LOGS  | Used to query information about all archived logs. |
| DV_CKPT_STATS  | Used to query statistical information related to checkpoints. |
