# Cluster Status Query

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:00:00.907Z pushedAt=2026-08-19T03:49:57.781Z -->

oGRAC allows you to view the status of the entire cluster, nodes, and cms through the cms utility, and confirm whether their running status is normal based on the query results. The prerequisite is to switch to the cluster user before execution.

## Command Description

Usage:

```
cms stat
cms stat -node [NODE_ID]
cms stat -res [RESOURCE_NAME]
cms stat -server [SERVER_ID]
```

Options:

| Option          | Description                |
| ------------- | ----------------- |
| NODE_ID       | Node ID              |
| RESOURCE_NAME | Resource name. Currently only db and dss resources are available. |
| SERVER_ID     | CMS node ID           |

## Command Reference

1. Query the cluster status.

```
cms stat
NODE_ID  NAME      STAT    PRE_STAT    TARGET_STAT   WORK_STAT   SESSION_ID   INSTANCE_ID   ROLE     LAST_CHECK              HB_TIME                 STAT_CHANGE
      0  db        ONLINE  OFFLINE     ONLINE                1            0             0   REFORMER 2025-10-28 11:19:44.427 2025-10-28 11:19:44.427 2025-10-27 20:57:51.886
      1  db        ONLINE  OFFLINE     ONLINE                1            0             1            2025-10-28 11:19:46.179 2025-10-28 11:19:46.179 2025-10-27 20:58:33.151
      0  dss       ONLINE  OFFLINE     ONLINE                1            1             0   REFORMER 2025-10-28 11:19:45.789 2025-10-28 11:19:45.789 2025-10-27 14:25:12.349
      1  dss       ONLINE  OFFLINE     ONLINE                1            1             1            2025-10-28 11:19:45.444 2025-10-28 11:19:45.444 2025-10-27 17:19:20.978
```

Result description:

| Field       | Meaning                  | Field Value                                                                                                  |
| ----------- | ------------------- | ---------------------------------------------------------------------------------------------------- |
| NODE_ID     | Node ID                | -                                                                                                    |
| NAME        | Node name           | db<br />dss                                                                                          |
| STAT        | Current node status         | ONLINE<br />OFFLINE<br />UNKNOWN                                                          |
| PRE_STAT    | Previous node status        | -                                                                                                    |
| TARGET_STAT | Next target status of the node      | -                                                                                                    |
| WORK_STAT   | Node working status         | RC_JOINING (0): joining the cluster<br />RC_JOINED (1): joined the cluster<br />RC_LEAVING (2): leaving the cluster<br />RC_LEFT (3): left the cluster |
| SESSION_ID  | Node ID           | When there is only one resource, the ID is 0; when the resource is offline, it is -1                                                                                |
| INSTANCE_ID | Instance ID of the node        | -                                                                                                    |
| ROLE        | Role of the node          | Can be REFORMER or non-REFORMER (shown as blank). Each resource type has exactly one reformer per cluster.                                               |
| LAST_CHECK  | Time of the last check on the node     | Equal to HB_TIME. When the node is offline, the time is 2000-01-01 00:00:00.000.                                                   |
| HB_TIME     | Time when the last heartbeat message from the node was received | When the node is offline, the time is 2000-01-01 00:00:00.000.                                                               |
| STAT_CHANGE | Time of the last node status change   | -                                                                                                    |

**NOTE**

`LAST_CHECK` and `HB_TIME` are compared, which can generally be used to observe whether the cms server process is normal. If an array fault or array communication fault occurs, it can be observed that `LAST_CHECK` is updated while `HB_TIME` does not change.

2. Query node status.

View status of all nodes:

```
cms stat -node
NODE_ID   NODE_NAME                       ROLE          
      0   node0                           server        
      1   node1                           agent  
```

View status of a specified node:

```
cms stat -node 0
NODE_ID   NODE_NAME                       ROLE          
      0   node0                           server 
```

Result description:

| Field       | Meaning                                                                                               |
| --------- | ------------------------------------------------------------------------------------------------ |
| NODE_ID   | Node ID                                                                                            |
| NODE_NAME | Node name                                                                                            |
| ROLE      | Node type: <br />When the node CMS is the Master, the node type is server, and the CMS Master is responsible for controlling processes such as split-brain arbitration.<br />When the node CMS is not the Master, the node type is agent. |

3. Query resource status.

View status of all resources:

```
cms stat -res
NODE_ID  RESOURCE_NAME  STAT   PRE_STAT  TARGET_STAT  WORK_STAT                      
0           db                ONLINE  OFFLINE  ONLINE        1
1           db                OFFLINE UNKNOWN  UNKNOWN       0
LAST_CHECK            STAT_CHANGE
2025-10-28 17:02:05.154    2025-10-28 16:30:10.262 
2000-01-01 00:00:00.000    2025-10-22 16:33:36.612 
```

View specified db resource status:

```
cms stat -res db
NODE_ID  RESOURCE_NAME  STAT   PRE_STAT  TARGET_STAT  WORK_STAT                      
0           db                ONLINE  OFFLINE  ONLINE        1
1           db                OFFLINE UNKNOWN  UNKNOWN       0
LAST_CHECK            STAT_CHANGE
2025-10-28 17:05:01.083    2025-10-28 16:30:10.262 
2000-01-01 00:00:00.000    2025-10-22 16:33:36.612 
```

View specified dss resource status:

```
cms stat -res dss
NODE_ID  RESOURCE_NAME  STAT   PRE_STAT  TARGET_STAT  WORK_STAT
0        dss            ONLINE  OFFLINE  ONLINE       1
1        dss            OFFLINE UNKNOWN  UNKNOWN      0
LAST_CHECK                 STAT_CHANGE
2025-10-22 20:58:08.909    2025-10-22 19:57:58.712
2025-10-22 19:26:49.315    2025-10-22 19:26:50.395
```

Result description:

| Field          | Meaning                                                                                                                          |
| ------------- | --------------------------------------------------------------------------------------------------------------------------- |
| NODE_ID       | Node ID                                                                                                                       |
| RESOURCE_NAME | Resource name. Currently only db and dss resources are available.                                                                                                            |
| STAT          | Resource status, which can be:<br>- ONLINE<br>- OFFLINE<br>- UNKNOWN                                                              |
| PRE_STAT      | Previous status of the resource                                                                                                                   |
| TARGET_STAT   | Next target status of the resource                                                                                                                 |
| WORK_STAT     | Working status of the resource, which can be:<br>- RC_JOINING (0): joining the cluster<br>- RC_JOINED (1): joined the cluster<br>- RC_LEAVING (2): leaving the cluster<br>- RC_LEFT (3): left the cluster |
| LAST_CHECK    | Time of the last resource check. When the resource is offline, the time is 2000-01-01 00:00:00.000.                                                                      |
| STAT_CHANGE   | Time of the last resource status change                                                                                                               |

4. Query CMS status.

View the CMS status of all nodes:

```

cms stat -server
NODE_ID  SRV_READY  SEND_QUE  RECV_QUE  TIME_GAP(ms)
0        TRUE       0         0         0
1        TRUE       0         0         1

```

View the CMS status of a specified node:

```

cms stat -server 0
NODE_ID  SRV_READY   SEND_QUE  RECV_QUE  TIME_GAP(ms)
0        TRUE        0         0         0

```

Result description:

| Field     | Meaning                                                 |
| --------- | ------------------------------------------------------- |
| NODE ID   | Node ID                                                 |
| SRV_READY | CMS working status:<br />TRUE: CMS is working normally.<br />FALSE: CMS is paused. |
| SEND_QUE  | Number of messages in the CMS message sending queue                                |
| RECV_QUE  | Number of messages in the CMS message receiving queue                                |
| TIME_GAP  | Maximum time jump between the CMS on this node and the CMS on other nodes           |
