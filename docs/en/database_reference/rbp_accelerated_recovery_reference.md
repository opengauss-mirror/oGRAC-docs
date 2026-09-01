# RBP Accelerated Recovery Reference

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:01:45.070Z pushedAt=2026-08-19T02:09:45.787Z -->

## Overview

RBP is used to reuse pages saved in RBPS during database recovery, thereby shortening the recovery time in scenarios such as node fault recovery. This document describes how to configure RBP accelerated recovery on the database side.

For RBPS server-side configuration, CMS management, and the `rbps_ctl` command, see [RBPS Instructions](../tool_and_commandreference/server_tool/rbps_instructions.md). For the meaning, value range, default value, and effective mode of RBP-related GUC parameters, see the RBP parameter description in [Database Reference](./database_reference.md).

## Configuration Prerequisites

- RBPS has been deployed on the node that hosts the page cache, and the RBPS listening address and port have been confirmed.

- The database node number, the local RBP communication address, and the network connectivity from the database node to the RBPS listening port have been confirmed.

- RBP communication can use a NIC different from that used for service access. In production environments, it is recommended to configure a dedicated NIC or a dedicated network for RBP/RBPS traffic to reduce the impact of accelerated recovery traffic on the service network.

- In a two-node DTC deployment, the database side is typically configured to connect to the peer RBPS. Confirm the RBPS address that each database node needs to connect to based on the actual deployment relationship.

- After modifying basic RTO or RBP restart‑class parameters, you need to restart the database instance.

## Recommended Configuration

### Accelerated Recovery Configuration

In a two-node DTC scenario, the database side is typically configured to connect to the peer RBPS. In the following example, RBP uses an independent network address:

```ini
USE_RBP=TRUE
RBP_FOR_RECOVERY=TRUE               
RBP_PORT=2611            
RBP_RT_ANALYSIS=TRUE              
RBP_RT_PARSE_WORKERS=2            
RBP_RT_PAGE_OWNER_WORKERS=4        
RBP_ASSEMBLE_MAX_SCAN=300     
RBP_IP=node1_rbp_ip,node0_rbp_ip    
```

Here, `node*_rbp_ip` can be the RBP communication NIC address independent of the service access address. It is recommended to separate RBP/RBPS traffic from service traffic to prevent network traffic during page write, read, and recovery phases from affecting service access.

## Database Parameter Configuration

Parameters can be modified through the database configuration file or `ALTER SYSTEM`. When using `ALTER SYSTEM`, the command must be executed separately on each database node.

### Basic RTO Parameter Configuration

In addition to RBP parameters, it is recommended to configure the following basic database RTO parameters in accelerated recovery scenarios:

```sql
ALTER SYSTEM SET BUF_POOL_NUM = 128;
ALTER SYSTEM SET _UNDO_ACTIVE_SEGMENTS = 1024;
ALTER SYSTEM SET UNDO_RETENTION_TIME = 60;
ALTER SYSTEM SET ENABLE_MERGE_JOIN = FALSE;
ALTER SYSTEM SET JOB_THREADS = 0;
ALTER SYSTEM SET SQL_STAT = FALSE;
ALTER SYSTEM SET TIMED_STATS = FALSE;
ALTER SYSTEM SET STATS_LEVEL = 'BASIC';
```

### RBP Parameter Configuration

For RBP restart-class parameters, it is recommended to write them into the parameter file and then restart the instance:

```sql
ALTER SYSTEM SET USE_RBP = TRUE;
ALTER SYSTEM SET RBP_FOR_RECOVERY = TRUE;
ALTER SYSTEM SET RBP_IP = 'node1_rbp_ip,node0_rbp_ip';
ALTER SYSTEM SET RBP_PORT = 2611;
ALTER SYSTEM SET RBP_RT_ANALYSIS = TRUE;
ALTER SYSTEM SET RBP_RT_PARSE_WORKERS = 2;
ALTER SYSTEM SET RBP_RT_PAGE_OWNER_WORKERS = 4;
ALTER SYSTEM SET RBP_ASSEMBLE_MAX_SCAN = 300;
```

After the configuration is complete, restart the database instance and confirm that RBPS has started.

## Configuration Check

On the database side, you can first check whether the parameters have been loaded:

```sql
SHOW PARAMETER USE_RBP;
SHOW PARAMETER RBP_IP;
SHOW PARAMETER RBP_PORT;
```

Then check the RBPS server status and the recovery window:

```bash
cms stat -res rbps
rbps_ctl stats
rbps_ctl window
```

If RBP connection or heartbeat information exists in the database logs, it indicates that the basic link between the database side and RBPS has been established. Whether RBP is actually used during the recovery phase is still automatically determined by the database based on the recovery window and security check results.

## Common Configuration Issues

### Database Fails to Connect to RBPS After Restart

- Confirm that restart-class parameters have been written to the parameter file and the instance has been restarted.

- Confirm that the database node can access the RBPS listening port configured in `RBP_IP`.

- Confirm that `RBP_PORT` is consistent with `PORT` in `rbps.conf`.

### RBP Is Not Used During the Recovery Phase

- Confirm that `USE_RBP` and `RBP_FOR_RECOVERY` are enabled and take effect after restart.

- Confirm that `RBP_IP` is configured according to the actual peer relationship.

- Confirm that a window available for recovery exists in `rbps_ctl window`.

- Check the database logs for information such as connection failure, mismatched address configuration, or fallback to conventional recovery.

Even if the database side is configured correctly, if the current recovery window does not meet the usage conditions, the database still automatically falls back to conventional recovery.

### Runtime Analysis Not Started

- Confirm that `RBP_RT_ANALYSIS` is set to `TRUE` and the instance has been restarted.

- Confirm that the current environment belongs to an accelerated recovery scenario that supports runtime analysis.

- If resource usage is high, you can keep `RBP_RT_ANALYSIS=FALSE` and use only the basic RBP accelerated recovery path.
