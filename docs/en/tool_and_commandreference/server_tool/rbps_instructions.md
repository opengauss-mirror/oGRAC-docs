# RBPS Usage Instructions

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:51:12.314Z pushedAt=2026-08-17T00:46:29.699Z -->

## Overview

`rbps` is the page cache server of RBP. When RBP is enabled, the database can use pages stored in RBPS during the recovery process, thereby reducing the recovery time in node failure recovery scenarios.

RBPS is positioned as follows:

- A recovery acceleration component, not a data file, log file, or backup tool.
- It does not change the way users use SQL. Once enabled, page writes and reads are automatically completed by the database.
- If RBPS is unavailable, the RBP window does not meet the recovery conditions, or the recovery security check fails, the database will fall back to conventional recovery.
- In a CMS deployment, it is recommended that the `rbps` resource be managed primarily through CMS; `rbps_ctl` is mainly used for local auxiliary checks and fault location.

## Installation and Configuration

After a normal oGRAC installation, RBPS-related files are located in the database installation directory:

| File | Default Location | Description |
| --- | --- | --- |
| `rbps` | `$OGDB_HOME/bin/rbps` | RBPS server program. |
| `rbps_ctl` | `$OGDB_HOME/bin/rbps_ctl` | RBPS local auxiliary management command, used for starting/stopping, status checking, and management port query. |
| `rbps_contrl.sh` | `$OGDB_HOME/bin/rbps_contrl.sh` | Script invoked when CMS manages the `rbps` resource. |
| `rbps.conf` | `$OGDB_DATA/cfg/rbps.conf` | RBPS server configuration file. |

`rbps_ctl` reads `$OGDB_DATA/cfg/rbps.conf` by default, and a configuration file can also be specified using `-c`.

## Configuring rbps.conf

Each node where RBPS is deployed needs to configure `rbps.conf`. Common configurations are as follows:

```ini
HOST=0.0.0.0
PORT=2611

ADMIN_HOST=127.0.0.1
ADMIN_PORT=2711

LOG_FILE=$OGDB_HOME/../../log/rbps/rbps.log
PID_FILE=$OGDB_HOME/run/rbps.pid

MAX_CACHE_PAGES=3000000
CAPACITY_EVICT_ON_WRITE=false
READ_END_MODE=async
READ_PHASE_TIMEOUT=3

VERBOSE=false
TIMING_DIAG=false
```

Configuration item description:

| Name | Default Value | Description |
| --- | --- | --- |
| `HOST` | `0.0.0.0` | RBPS listening address. The database instance writes and reads pages through this address. |
| `PORT` | `2611` | RBPS listening port, which must be consistent with the connection port configuration on the database side. |
| `ADMIN_HOST` | `127.0.0.1` | Management port listening address. Commands such as `rbps_ctl stats` and `rbps_ctl window` use this address. |
| `ADMIN_PORT` | `2711` | Management port. Setting it to an unavailable port will cause management queries to fail. |
| `LOG_FILE` | `$OGDB_HOME/../../log/rbps/rbps.log` | RBPS log file. |
| `PID_FILE` | `$OGDB_HOME/run/rbps.pid` | RBPS process PID file. |
| `MAX_CACHE_PAGES` | `3000000` | Maximum number of cached pages. `0` means no limit. |
| `CAPACITY_EVICT_ON_WRITE` | `false` | Behavior when the cache reaches its limit. `false` means new pages are rejected when the limit is reached, and `true` means writing is allowed to continue while old pages are evicted in the background. |
| `READ_END_MODE` | `async` | Method for ending the recovery read phase. Possible values are `async` or `sync`. It is recommended to keep the default value. |
| `READ_PHASE_TIMEOUT` | `3` | Idle timeout in seconds for automatic release of orphan READ_PHASE. `0` means automatic release is disabled. |
| `VERBOSE` | `false` | Whether to output more detailed logs. |
| `TIMING_DIAG` | `false` | Whether to output time-consuming diagnostic logs. |

## Normal Deployment Process

### 1. Preparing the RBPS Server Configuration

On each RBPS node, verify that `HOST`, `PORT`, and `LOG_FILE` in `rbps.conf` are correct. If CMS is used for management, confirm that `$OGDB_HOME/bin/rbps_contrl.sh` and `$OGDB_HOME/bin/rbps_ctl` are executable.

### 2. Confirming Database-Side Configuration

On each database node, complete the database-side configuration as described in [RBP Accelerated Recovery Reference](../../database_reference/rbp_accelerated_recovery_reference.md), and ensure that the database-side connection address and port match the RBPS server listening configuration. The database GUC parameter descriptions are not elaborated here.

### 3. Starting RBPS via CMS

During CMS deployment, the installation process attempts to pre-register the `rbps` resource. After RBP is enabled on the database side, RBPS can be started and checked using CMS:

```bash
su -s /bin/bash ograc
source ~/.bashrc

cms res -list
cms res -start rbps
cms stat -res rbps
```

Stop RBPS:

```bash
cms res -stop rbps
```

If RBP is disabled on the database side, CMS considers the `rbps` resource as disabled:

- `cms res -list` and `cms stat` will hide `rbps`.
- `cms res -start rbps` will fail with a message indicating that the resource is disabled.

### 4. Restarting the Database Instance

After modifying the restart-related parameters according to the database-side configuration reference, the database instance must be restarted. Once the instance is started, the database loads the RBP configuration and connects to RBPS. Under normal circumstances, RBP connection or heartbeat-related information can be observed in the logs.

## Verifying That Components Are Normal

It is recommended to check in the following order:

```bash
# 1. Check whether CMS has registered and manages the rbps resource.
cms res -list

# 2. Check whether rbps is online from the CMS perspective. If STAT shows ONLINE, CMS considers the RBPS resource online.
cms stat -res rbps 

# 3. Check whether the local process and management port are normal.
rbps_ctl status     # "rbps is running" indicates that the local RBPS process has started.
rbps_ctl stats      # "OK" indicates that the local RBPS management port is accessible.
```

## rbps_ctl Auxiliary Command Reference

`rbps_ctl` is not the preferred start/stop entry point under CMS deployment. For routine management, `cms res -start rbps`, `cms res -stop rbps`, and `cms stat -res rbps` should be used first. The commands in this section are used when it is necessary to locally verify the process, check the management port, view cache statistics, or inspect the recovery window.

### start

Starts RBPS locally. In a CMS deployment, it is recommended to use `cms res -start rbps` for routine startup.

Example output upon successful startup:

```text
rbps started: pid=12345 listen=0.0.0.0:2611 admin=127.0.0.1:2711 conf=/data/ograc/cfg/rbps.conf log=/data/ograc/log/rbps/rbps.log
```

Example output if the service is already running:

```text
rbps is already running: pid=12345 conf=/data/ograc/cfg/rbps.conf
```

### status

Queries the RBPS process status.

Example output when running:

```text
rbps is running: pid=12345 listen=0.0.0.0:2611 admin=127.0.0.1:2711 conf=/data/ograc/cfg/rbps.conf log=/data/ograc/log/rbps/rbps.log
```

Example output when not running:

```text
rbps is not running
```

### stop and stop_force

`stop` stops RBPS normally on the local node. `stop_force` first attempts a normal stop; and if the process does not exit within a short period, it forcibly terminates the process. In a CMS deployment, it is recommended to use `cms res -stop rbps` for routine stopping. `stop_force` is recommended only when CMS fails to stop the resource normally or during local troubleshooting.

Output example:

```text
rbps stopped: pid=12345
rbps force stopped: pid=12345
```

### restart

Restarts RBPS locally, which is equivalent to executing `stop` first and then `start`.

### stats

Queries brief RBPS statistics.

Output example:

```text
OK cache_total=100 pending_total=0 max_cache_pages=0 capacity_evict_on_write=0
```

Field description:

| Field | Description |
| --- | --- |
| `OK` | The management query is executed successfully. |
| `cache_total` | Total number of pages currently in the RBPS cache. |
| `pending_total` | Reserved field, indicating the total number of pages to be returned during the current recovery read phase. |
| `max_cache_pages` | Configured upper limit of cached pages. `0` indicates no limit. |
| `capacity_evict_on_write` | Whether eviction on write is allowed when the cache limit is reached. `0` indicates no, and `1` indicates yes. |

### window

Queries the current RBP recovery window.

Output example:

```text
WINDOW_BEGIN
cache_pages=100
pending_pages=0
max_cache_pages=0
capacity_evict_on_write=0
max_lsn=123456789
begin=rst=0 asn=1 blk=100 lfn=1000 lsn=123000000
rcy=rst=0 asn=1 blk=200 lfn=2000 lsn=123400000
lrp=rst=0 asn=1 blk=250 lfn=2500 lsn=123456789
evict_in_progress=0
purge_stable=1
empty_reason=
WINDOW_END
```

Field description:

| Field | Description |
| --- | --- |
| `cache_pages` | Number of currently cached pages. |
| `pending_pages` | Number of pages waiting to be returned in the current recovery read phase. |
| `max_lsn` | Maximum page LSN among the currently cached pages in RBPS. |
| `begin` | Start point of the current window. If the recovery point is earlier than or not within the window, the database will not use this window for acceleration. |
| `rcy` | Recoverable point of the current window. When conditions are met, the database can use RBPS pages to shorten the recovery time. |
| `lrp` | Upper bound of log replay covered by the current window. |
| `evict_in_progress` | Whether cache eviction is in progress. |
| `purge_stable` | Whether the window is stable after purging. |
| `empty_reason` | Reason why the window is empty. |

### read_phase

Queries the status of the read recovery phase.

Output example:

```text
READ_PHASE_BEGIN
active=0
ending=0
elapsed_ms=0
idle_ms=0
inflight_reads=0
timeout_s=3
dropped_page_writes=0
timeout_warned=0
READ_PHASE_END
```

Field description:

| Field | Description |
| --- | --- |
| `active` | Whether the system is in the read recovery phase. `1` indicates yes. |
| `ending` | Whether READ_PHASE is ending. |
| `elapsed_ms` | Duration of the current READ_PHASE, in milliseconds. |
| `idle_ms` | Idle time of READ_PHASE, in milliseconds. |
| `inflight_reads` | Number of read requests currently being processed. |
| `timeout_s` | Automatic release timeout for READ_PHASE, in seconds. |
| `dropped_page_writes` | Number of write page requests discarded during READ_PHASE. |
| `timeout_warned` | Whether a timeout warning has been printed. |

### force_read_end

Forcibly releases residual READ_PHASE. Use this only when the recovery read phase is confirmed to be abnormally left over.

Output example:

```text
OK active_before=1 ending_before=0 cleared=1 elapsed_ms=5000 dropped_page_writes=10 detached_pages=20
```

Field description:

| Field | Description |
| --- | --- |
| `active_before` | Whether READ_PHASE was in the active state before the command was executed. |
| `ending_before` | Whether READ_PHASE was ending before the command was executed. |
| `cleared` | Whether the cleanup is completed. `1` indicates yes. |
| `elapsed_ms` | Duration of the cleaned READ_PHASE, in milliseconds. |
| `dropped_page_writes` | Number of write page requests discarded during READ_PHASE. |
| `detached_pages` | Number of pages released during cleanup. |

### exists

Queries whether a specified page is in the RBPS cache. The page format is `<file>-<page>`, and `<file>_<page>` or `<file>/<page>` can also be used.

Output example:

```text
FOUND file=1 page=100 lsn=123456789 pcn=10 cache_total=20
```

Output example when not found:

```text
NOT_FOUND file=1 page=100 cache_total=20
```

### dump

Outputs detailed diagnostic information for a specified page, primarily used for issue locating. If the page does not exist, the output is consistent with the `NOT_FOUND` result of `exists`.

Output example:

```text
rbps_ctl dump 12-14529
DUMP_BEGIN
file=12
page=14529
qid=1
pending=1
cache_total=59485
writer_inst=0
writer_seq=75433224
page_lsn=75433224
page_pcn=10
page_checksum=0x0000
trunc_rst=0
trunc_asn=1
trunc_blk=89775149
trunc_lfn=69383
trunc_lsn=2199023255552
lrp_rst=0
lrp_asn=1
lrp_blk=89776459
lrp_lfn=69410
lrp_lsn=2199023255552
reset=rst=0 asn=1 blk=89030039 lfn=50761 lsn=2199023255552
frontier=rst=0 asn=1 blk=91513030 lfn=102072 lsn=2199023255552
trunc_hex=010000002ddc590500001c3c040000000000000000020000
lrp_hex=010000004be159050000883c040000000000000000020000
page_header_hex=c13800000c00110200000000ff03010008057f04000000000a000000000000000c086c00003e01010170ab07540a000002b0a009540a0000090000000200ff3fc23800000c00501ca01f5003140014000200000002e01530a38b000000000000
page_tail_hex=90042c03c001600002e01530a38b00000c026a030100000001144a0615001000017076b77b0a0000050000000200000001c01e0417000000000000000a000000
DUMP_END
```

Field description:

| Field | Description |
| --- | --- |
| `DUMP_BEGIN`/`DUMP_END` | Start and end markers of the diagnostic information. |
| `file`/`page` | Data file number and page number. |
| `pending` | Whether the page is in the pending return set during the BATCH_READ recovery read phase. |
| `writer_inst` | Instance number that wrote this page. |
| `page_lsn`/`page_pcn` | LSN and PCN in the page header. |
| `page_header_hex`/`page_tail_hex` | Hexadecimal preview of the page header and page tail. |

### query

Sends a query command directly to the management port. Commonly used commands include:

```bash
rbps_ctl query STATS
rbps_ctl query WINDOW
rbps_ctl query READ_PHASE
rbps_ctl query FORCE_READ_END
```

The output fields are the same as those of `stats`, `window`, `read_phase`, and `force_read_end`, respectively.

## Troubleshooting

### rbps_ctl Reports That the Configuration File Cannot Be Found

Verify that the environment variables have been loaded, or specify the configuration file using `-c`:

```bash
source ~/.bashrc
rbps_ctl status -c $OGDB_DATA/cfg/rbps.conf
```

### CMS Fails to Start the rbps Resource

First, follow [RBP Accelerated Recovery Reference](../../database_reference/rbp_accelerated_recovery_reference.md) to confirm that the database has completed the configuration and restarted the instance, and then execute:

```bash
cms res -start rbps
cms stat -res rbps
```

### stats or window Returns No Result

Prioritize checking the following:

- Whether `cms stat -res rbps` shows the resource as `ONLINE`.
- If CMS displays an abnormal status, use `rbps_ctl status` to confirm whether the local process shows `rbps is running`.
- Whether `ADMIN_HOST` and `ADMIN_PORT` in `rbps.conf` are correct.
- Whether the current user has loaded environment variables such as `$OGDB_HOME` and `$OGDB_DATA`.

### RBP Not Used During Recovery Phase

Check in the following order:

- Whether the database has completed configuration according to [RBP Accelerated Recovery Reference](../../database_reference/rbp_accelerated_recovery_reference.md) and restarted the instance.
- Whether the RBPS service address and port match the connection configuration on the database side, and whether the service port is accessible from the database node.
- Whether `rbps_ctl window` has a valid window.
- Whether the database logs contain information such as connection failures, mismatched address configurations, or fallback to conventional recovery.

Even if RBP is enabled, the database will automatically fall back to conventional recovery if the current recovery window does not meet the usage conditions.

### READ_PHASE Not Released for a Long Time

First, query:

```bash
rbps_ctl read_phase
```

If it is confirmed that the recovery read phase has been abnormally left over, execute:

```bash
rbps_ctl force_read_end
```

### Cache Page Limit Reached

If `MAX_CACHE_PAGES` is set and `CAPACITY_EVICT_ON_WRITE=false`, new pages will be rejected for writing once the cache is full. You may increase `MAX_CACHE_PAGES` according to the environment capacity, or configure the following:

```ini
CAPACITY_EVICT_ON_WRITE=true
```

RBPS must be restarted after `rbps.conf` is modified.
