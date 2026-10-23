# oGRAC Performance Tuning Guide

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-10-13T02:00:07.474Z pushedAt=2026-10-19T07:38:17.578Z -->

## Introduction

This section mainly describes the test method for measuring the peak TPCC performance of oGRAC, as well as the hardware environment, database-related parameter tuning, and OS tuning required to achieve optimal tpmC performance.

## Hardware Environment

- Hardware environment requirements:
    |Resource Name|Configuration|
    |:--|:--|
    |oGRAC server|Kunpeng 920B 2\*80 core, 1 TiB DDR6 MEM, 2\*3 TiB NVME DISK, 2 \* high-performance NICs (see the third row for NIC configuration)|
    |Load testing client server|Kunpeng 920 2\*48 core (or higher), 1 TiB DDR6 MEM, 2\*3 TiB NVME DISK, 2 \* high-performance NICs (see the third row for NIC configuration)|
    |High-performance NIC|The NIC hardware supports at least 64 queues and RoCE. Recommended models: **Huawei high-performance NIC SP670/SP680/SP681**, at 25GE or higher. Each server is configured with 2 NICs, and each NIC has at least 2 ports.|
    |Storage array|Supports NVMe over RoCE/NVMe over Fabrics. Recommended model: **Huawei centralized enterprise storage array Dorado 18000 V6**. The NICs configured on the storage array must also support RoCE at 25GE or higher.|
    |Switch|Supports RoCE, 25GE/100GE, and lossless network configuration.|

## Software Environment

- Software environment requirements:
    |Resource Name|Version|
    |:--|:--|
    |Database|oGRAC 8.0.0 LTS |
    |TPCC client|[benchmarksql tool for oGRAC](https://gitcode.com/opengauss/oGRAC-common-tools)|
    |OS|openEuler 24.03-LTS SP1|
    |NIC driver|The latest version released by the Huawei Technical Support Community that matches the OS|

## Networking Mode

![Networking Reference](./figures/network_en.png)

## Storage Array Connectivity Configuration Guide

- oGRAC nodes can connect to the storage array through three protocols: iSCSI, FC, and NoF. For configuration, refer to the connectivity guide provided by the storage array vendor.
- After hardware and cable anomalies are ruled out, the theoretical I/O latency performance ranking is **NoF > FC > iSCSI**.
- Because NoF configuration is slightly more demanding than iSCSI and FC, a simple NoF configuration guide based on Huawei high-performance NICs, Huawei enterprise-class centralized storage arrays, and Kunpeng servers is provided here. Refer to [NoF Configuration Guide Based on Kunpeng Servers](./nof_configure_guide.md).

## Host BIOS and OS Tuning

1. Host BIOS parameter settings (the locations of configuration items may vary across BIOS versions; use the actual version as the reference)

    - BIOS$\rightarrow$Advanced$\rightarrow$MISC Configuration, set Support Smmu to Disabled
    - BIOS$\rightarrow$Advanced$\rightarrow$Memory Configuration, set Die Interleaving to Disable
    - BIOS$\rightarrow$Advanced$\rightarrow$Power And Performance Configuration, set Performance Profile to HPC
    - BIOS$\rightarrow$Advanced$\rightarrow$Power And Performance Configuration, set Power Policy to Performance Profile
    - BIOS$\rightarrow$Advanced$\rightarrow$Power And Performance Configuration$\rightarrow$CPU PM Control, set CPU Prefetching to Enabled
    - BIOS$\rightarrow$Advanced$\rightarrow$Power And Performance Configuration$\rightarrow$CPU PM Control, set SMT2 to Enabled
    - BIOS$\rightarrow$Advanced$\rightarrow$Power And Performance Configuration$\rightarrow$CPU PM Control, set TidCMP to Disabled

2. Operating system optimization

    - Disable irqbalance
    
        ```bash
        systemctl stop irqbalance
        ```

    - Adjust numa_balance

        ```bash
        echo 0 > /proc/sys/kernel/numa_balancing
        ```

    - Adjust transparent huge pages

        ```bash
        echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
        echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag
        ```

## Disk I/O Queue Scheduling Mechanism Settings
    
1. For the disk I/O queue scheduling mechanism settings, `sd*` here refers to the block device names on the host side to which the LUNs of the storage array are mapped.
    
    ```bash
    echo none > /sys/block/sd*/queue/scheduler
    ```
    
## Binding NIC Interrupts to CPU Cores

It is recommended to bind cores for the network ports connecting oGRAC nodes, the network ports connecting client nodes and oGRAC nodes, and the network ports connecting oGRAC nodes and the storage array. For details, see the following:

1. Set the number of queues of the NIC to the maximum.
    ```
    ethtool -l enp4s0

    ## Set the combined value to the maximum supported value shown in Pre-set maximums.
    ethtool -L enp4s0 combined 64
    ```
2. Obtain the interrupt number corresponding to the NIC.
    ```
    [root@localhost]# cat /proc/interrupts | grep enp4s0 | awk '{print $1$NF}'
    405:enp4s0_qp0
    406:enp4s0_qp1
    407:enp4s0_qp2
    408:enp4s0_qp3
    ...
    ```
3. Bind the corresponding CPU cores. Divide the maximum supported combined value of the NIC by the number of NUMA nodes on the server to obtain the number of CPUs to be selected from each NUMA node for binding. It is recommended to bind to the last n CPUs in the CPU range corresponding to each NUMA node. For example, if the maximum supported combined value of the NIC is 64 and the number of NUMA nodes on the server is 4, 16 CPUs are selected from each NUMA node for binding. The CPU ranges corresponding to the 4 NUMA nodes are 0-95, 96-191, 192-287, and 288-383 in sequence, so the CPU ranges to be bound are 80-95, 176-191, 272-287, and 368-383.
    ```
    [root@localhost]# echo 80 >  /proc/irq/405/smp_affinity_list
    [root@localhost]# echo 81 >  /proc/irq/406/smp_affinity_list
    [root@localhost]# echo 82 >  /proc/irq/407/smp_affinity_list
    [root@localhost]# echo 83 >  /proc/irq/408/smp_affinity_list
    ...
    ```
## Preparing Test Tools

Prepare the TPCC test client, and perform the operations on all load-testing client nodes.

1. Install the Git, JDK, and Ant dependency packages.
    
    ```bash
    [root@localhost]# yum install -y git ant java-11-openjdk
    ```

2. Download the TPCC test tool adapted for oGRAC.
    
    ```bash
    [root@localhost]# git clone https://gitcode.com/opengauss/oGRAC-common-tools.git
    ```

3. In the directory where BenchmarkSQL is located, run the `ant` command to compile it. After successful compilation, the `build` and `dist` directories are generated.
    
    ```bash
    [root@localhost]# cd /home/oGRAC-common-tools/benchmarksql_ograc
    [root@localhost]# ant
    Buildfile: /home/oGRAC-common-tools/benchmarksql_ograc/build.xml
    ...
    BUILD SUCCESSFUL
    Total time: 1 second
    ```

4. [Download the corresponding JDBC driver](https://opengauss.org/en/download/) to the `lib/postgresql` folder in the BenchmarkSQL directory, decompress it, and delete the built-in JDBC driver.
    
    ```bash
    [root@localhost]# pwd
    /home/oGRAC-common-tools/benchmarksql_ograc/lib/postgres
    [root@localhost]# ls
    openGauss-JDBC-7.0.0-RC3.tar.gz  postgresql-9.3-1102.jdbc41.jar
    [root@localhost]# rm -f postgresql-9.3-1102.jdbc41.jar
    [root@localhost]# tar -xf openGauss-JDBC-7.0.0-RC3.tar.gz
    [root@localhost]# ls
    openGauss-JDBC-7.0.0-RC3.tar.gz  postgresql.jar
    ```

5. Adjust the performance test related configurations.
- The tool already provides a configuration example `/home/oGRAC-common-tools/benchmarksql_ograc/run/ograc_run.og`. Modify the IP address and port in the connection string. `nodeId` indicates the ID of the load-testing node, and `nodeCnt` indicates the number of load-testing nodes.
- To test two nodes simultaneously, start two load-testing clients. In the test configuration used by one load-testing client, set `nodeId` to 0; in the test configuration used by the other load-testing client, set `nodeId` to 1.
    ```bash
    db=postgres
    driver=org.postgresql.Driver
    conn=jdbc:oGRAC://x.x.x.x:1661
    nodeId=0
    nodeCnt=2
    user=TPCC
    password=XXXXXXX
    warehouses=2000
    loadWorkers=100
    terminals=1000
    runTxnsPerTerminal=0
    runMins=15
    limitTxnsPerMin=0
    terminalWarehouseFixed=true
    newOrderWeight=45
    paymentWeight=43
    orderStatusWeight=4
    deliveryWeight=4
    stockLevelWeight=4
    ```

## Installing the oGRAC Database

1. It is recommended to prepare five LUN devices as shown in the following example and mount them to the two oGRAC nodes to be installed. The NoF networking is used as an example here.

    |LUN Purpose|Size|DEV Name|
    |:--|:--|:--|
    |GCC DISK|5GB|/dev/nvme2n1 |
    |DATA DISK|4TB|/dev/nvme2n2 |
    |NODE 0 REDO DISK|2TB|/dev/nvme2n3|
    |NODE 1 REDO DISK|2TB|/dev/nvme2n4 |
    |REDO ARCHIVE DISK|2TB|/dev/nvme2n5 |

2. Prepare the installation configuration file `config_params_lun.json` with the following content (when installing on the other node, only change node_id to 1, and it is recommended that redo_size be at least 150G):

    ```json
    {
        "deploy_mode": "dss",
        "node_id": "0",
        "cms_ip": "1.1.1.1;1.1.1.2",
        "db_type": "1",
        "mes_ssl_switch": false,
        "MAX_ARCH_FILES_SIZE": "300G",
        "redo_num": "3",
        "redo_size": "200G",
        "auto_tune": "0",
        "dss_vg_list": {
            "vg1": "/dev/nvme2n2",
            "vg2": "/dev/nvme2n3",
            "vg3": "/dev/nvme2n5",
            "vg4": "/dev/nvme2n4"
        },
        "gcc_home": "/dev/nvme2n1",
        "cms_port": "14587",
        "dss_port": "1811",
        "ograc_port": "1611",
        "interconnect_port": "1601,1602",
        "_SHM_KEY": 17,
        "module_config": {
            "ograc_home": "/home/ograc/app",
            "data_root": "/home/ograc/dbdata",
            "user": "ograc"
        },
        "install_type": "override"
    }
    ```
3. Install the two-node oGRAC cluster environment. For details, see [oGRAC Two-Node Deployment Guide](https://docs.opengauss.org/en/docs/latest/ograc/installation_guide/two_nodes_guide/ograc_two_node_installation.html).


## Setting Database Parameters for Performance Testing (for Extreme Performance Testing)

Prerequisite: The oGRAC two-node cluster has been installed successfully.

1. Switch to the database installation user and run the following commands on node 0 and node 1 respectively.
    ```sql
    # Connect to the database.
    ogsql / as sysdba -q

    # Execute the following SQL.
    alter system set ENABLE_SYSDBA_REMOTE_LOGIN = TRUE;
    alter system set ENABLE_SYS_REMOTE_LOGIN = TRUE;
    alter system set _OPTIM_SUBQUERY_REWRITE = true;
    alter system set CHECKPOINT_PERIOD = 60;
    alter system set CHECKPOINT_PAGES = 2000000;
    alter system set LOG_BUFFER_COUNT = 128;
    alter system set _UNDO_ACTIVE_SEGMENTS = 1024;
    alter system set UNDO_RETENTION_TIME = 60;
    alter system set DATA_BUFFER_SIZE = '768G';
    alter system set ENABLE_MERGE_JOIN = false;
    alter system set TIMED_STATS = false;
    alter system set SQL_STAT = false;
    alter system set STATS_LEVEL = basic;
    alter system set JOB_QUEUE_PROCESSES = 0;
    alter system set JOB_THREADS = 0;

    ## The IP address to be listened on is configured here. Configure the IP address that is reachable between the client node and the oGRAC nodes.
    alter system add LSNR_ADDR 'x.x.x.x';
    alter system add hba entry 'host * 0.0.0.0/0';
    alter system reload hba config;

    alter system set BUF_POOL_NUM = 128;
    alter system set UNDO_RESERVE_SIZE = 512;
    alter system set _UNDO_AUTO_SHRINK = TRUE;
    alter system set _UNDO_PERF_PREALLOC = TRUE;
    alter system set _UNDO_PREALLOC_PAGES = 4000;

    alter system set DTC_CKPT_NOTIFY_TASK_RATIO = 0.125;
    alter system set DTC_CLEAN_EDP_TASK_RATIO = 0.125;
    alter system set DTC_TXN_INFO_TASK_RATIO = 0.25;
    alter system set AUDIT_LEVEL = 0;

    ## The CPU range is configured here. Configure it based on the actual CPU range of the server (which can be determined using the system command lscpu). The configured range is obtained by subtracting the first 2 cores and the last 8 cores from the CPU range corresponding to each NUMA node. For example, if the CPU range corresponding to numa0 is 0-79, the configured range is 2-71, and so on.
    alter system set CPU_GROUP_INFO = '2-71 82-151 162-231 242-311';
    ```
2. Restart the databases on node 0 and node 1 respectively.
   ```bash
    cms res -stop db -node 0
    cms res -start dss -node 0
    cms res -start db -node 0
    cms res -stop db -node 1
    cms res -start dss -node 1
    cms res -start db -node 1
   ```

## Creating the Test Database and User
    
1. Create the database `tpcc` and the user `tpcc` on node 0.
    ```sql
    #Connect to the database
    ogsql / as sysdba -q

    #Execute the following SQL
    create user TPCC identified by 'xxxxxxx';
    grant create session to TPCC;
    grant create table to TPCC;
    grant dba to TPCC;
    grant inherit privileges on user SYS to TPCC;
    ```

## Pre-Expanding Data Files

Pre-expanding data files can avoid performance fluctuations caused by data file expansion during performance stress testing. When testing 1000-warehouse data, it is recommended to pre-expand userX.dat to 200G; when testing 2000-warehouse data, it is recommended to pre-expand userX.dat to 500G.

1. Perform the following operations on node 0.
    ```sql
    # Connect to the database.
    ogsql / as sysdba -q

    # Execute the following SQL statements.
    alter database datafile '+vg1/undo01.dat' resize 32G;
    alter database datafile '+vg1/undo11.dat' resize 32G;
    alter database datafile '+vg1/undo02.dat' resize 32G;
    alter database datafile '+vg1/undo12.dat' resize 32G;
    alter database datafile '+vg1/user1.dat' resize 500G;
    alter database datafile '+vg1/user2.dat' resize 500G;
    alter database datafile '+vg1/sysaux' resize 10G;
    ```

## Modifying the Test Client Configuration
    
1. Go to the run folder in the BenchmarkSQL directory, edit the benchmarksql configuration file, and modify the test parameters, including the database username, password, IP, port, and database.
    
    ```bash
    [root@localhost]# cd /home/oGRAC-common-tools/benchmarksql_ograc/run/
    [root@localhost]# vim ograc_run.og
    db=postgres
    driver=org.postgresql.Driver
    conn=jdbc:oGRAC://x.x.x.x:1661 ## IP through which the oGRAC node connects to the client node
    nodeId=0 ## Node ID
    nodeCnt=2 ## Total number of nodes
    user=TPCC
    password=XXXXXX
    warehouses=2000
    loadWorkers=100
    terminals=1000
    runTxnsPerTerminal=0
    runMins=15
    limitTxnsPerMin=0
    terminalWarehouseFixed=true
    newOrderWeight=45
    paymentWeight=43
    orderStatusWeight=4
    deliveryWeight=4
    stockLevelWeight=4
    ```

## Importing Test Data
    
1. Execute the following command to import the test data.

    ```bash
    [root@localhost]# sh runDatabaseBuild.sh ograc_run.og
    ```

## Executing the Test

1. Execute the following command to start the test.
    ```bash
    [root@localhost]# sh runBenchmark.sh ograc_run.og
    ```
2. To test the multi-write performance of two nodes, start two TPCC tests simultaneously on the load-testing client node 0 connected to oGRAC node 0 and the load-testing client node 1 connected to oGRAC node 1 (change `nodeId` to 1 in the `ograc_run.og` file of node 1).

## Observing System Resource Tools

- Use htop to observe CPU usage.
- Use iostat to view system IO usage: `iostat -xdk 1`
- Use sar to view network usage: `sar -n DEV 1`
- Monitor oGRAC wait events by executing SQL: `ogsql / as sysdba -q -c "select EVENT,count(sid) from DV_SESSION_WAITS where WAIT_TIME_MIRCO > 0 group by EVENT;"`
  
  Under normal circumstances, the wait event that consumes the most time is `log file sync`, because a large number of threads are concurrently waiting for REDO logs to be flushed to disk. If this is not the case, analyze the actual wait events.

## Test Result Reference

- On Kunpeng 920B servers (320 cores) with OceanStor Dorado 18000 V6 centralized storage, the reference results of the TPCC test executed according to the preceding steps are as follows. The test data volume is 2000 warehouses, with a concurrency level of 1000 per node:

    |Node Name|Warehouses|Concurrency|tpmC(NewOrders)|
    |:--|:--|:--|:--|
    |Node 0| 2000 | 1000 | 200w |
    |Node 1| 2000 | 1000 | 192w |
