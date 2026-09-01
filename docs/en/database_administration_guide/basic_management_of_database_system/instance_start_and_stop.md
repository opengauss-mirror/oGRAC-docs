# Instance Start and Stop

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:00:03.674Z pushedAt=2026-08-20T03:49:48.995Z -->

During cluster maintenance, troubleshooting, and repair, you can start and stop the cluster and instances as needed.

All CMS commands require the cluster user context. Switch to the cluster user using the command below:

```
# su -s /bin/bash - ograc
```

## Command Description

Usage:

```
cms res -start
cms res -start RESOURCE_NAME
cms res -start RESOURCE_NAME -node NODE_ID
cms res -start RESOURCE_NAME TIMEOUT
cms res -stop RESOURCE_NAME
cms res -stop RESOURCE_NAME -node NODE_ID
```

Options:

| Option        | Description               |
| ------------- | ------------------------- |
| RESOURCE_NAME | Resource name. Currently only the db and dss resources are available.         |
| NODE_ID       | Node ID, which can be queried using the `cms stat -node` command. |
| TIMEOUT       | Startup timeout threshold. If the startup time exceeds the threshold, the command returns; otherwise, it waits. The default value is 600 seconds. |

## Command Reference

1. Start the cluster or instance.

    **NOTE**

    - Before running this command, check whether the CMS server on the node to be started is alive (run the `ps -ef|grep 'cms server'` command to check whether a cms server process exists).

    - When starting resources on a specified node, the default wait time is 600 seconds. If the resources are still not started successfully after 600 seconds, the command times out and exits with an error.

    Start the cms resource (to mount in the background, use `cms server -start &`):

    ```
    cms server -start
    start resource succeed.
    NODE_ID     = 0
    CMS_HOME    = /opt/oGRAC/data
    GCC_HOME    = /opt/oGRAC/data/gcc_home/gcc_file
    CMS GCC_TYPE:FILEVERSION     = oGRAC Debug 25.12 0d24758
    cms startup...
    MES: LSNR 127.0.0.1:23456
    ```

    Start all db nodes in the cluster:

    ```
    cms res -start db
    start resource succeed.
    ```

    Start the specified db node:

    ```
    cms res -start db -node 0
    start resource succeed.
    ```

    Start all dss nodes in the cluster:

    ```
    cms res -start dss
    start resource succeed.
    ```

    Start the specified dss node in the cluster:

    ```
    cms res -start dss -node 0
    start resource succeed.
    ```

    Start the cluster nodes with a specified wait time:

    ```
    cms res -start db 120000
    start resource succeed.
    ```

2. Stop the cluster or instance.

    Stop the cms resource:

    ```
    cms server -stop
    stop cms server succeed.
    ```

    Stop all db nodes in the cluster:

    ```
    cms res -stop db
    stop resource succeed.
    ```

    Stop the specified db node:

    ```
    cms res -stop db -node 0
    stop resource succeed.
    ```

    Stop all dss nodes in the cluster:

    ```
    cms res -stop dss
    stop resource succeed.
    ```

    Stop the specified dss node:

    ```
    cms res -stop dss -node 0
    stop resource succeed.
    ```

> [!WARNING]
>
> * DSS Introduction
>
> Distributed Storage Service (DSS) is the distributed shared storage service of oGRAC, responsible for managing shared disk volume groups (VGs). Its core responsibility is to provide access to shared storage between nodes, manage DSS disk groups (such as vg1 and vg2), and provide the underlying I/O path for data files and log files. DSS runs through processes such as dssserver, and its key resources are DSS VGs mapped from raw devices or block devices, which can be viewed through `dsscmd lsvg`. DSS itself does not parse SQL or participate in transaction processing; it only provides block-level or file-level storage services and belongs to the infrastructure layer.
>
> * DB Introduction
>
> DB is the oGRAC database instance (ogracd), responsible for core database functions such as SQL parsing, transaction processing, and cache management. It runs business SQL, manages the buffer pool, executes DML/DDL operations, and maintains cluster status under CM control. The main process of DB is ogracd, and its key resources include tablespaces, data files, and WAL logs. It should be noted that although DB manages these logical resources, all actual data persistence operations are completed by the underlying DSS.
>
> * Relationship Between the Two
>
> DB and DSS have a consumer‑foundation relationship. The data files and archive files of DB are all located in the VGs managed by DSS. DSS is the storage foundation of DB, and DB is the consumer of DSS. In terms of startup order, DB must be able to normally access the DSS VGs when starting; otherwise, DB cannot start. Therefore, the strict order of "DSS first, DB second" must be followed. At the communication level, DB interacts with dssserver through the DSS client library to execute specific I/O requests. In terms of fault impact, once DSS fails, DB will immediately fail or hang due to the disconnection of the I/O path; whereas a DB fault usually does not directly affect the DSS process, but it will trigger cascading behavior during stop operations. If the CMS Server process is still alive, the dssserver process will be automatically restarted. Therefore, it is recommended to use `ps ux | grep dssserver | grep -v grep` to confirm whether the dssserver process is alive before starting DB. In terms of functional positioning, DSS belongs to the infrastructure layer, providing only block-level or file-level storage services and not parsing SQL at all; DB belongs to the application service layer, responsible for SQL processing, transaction management, and cache control.
