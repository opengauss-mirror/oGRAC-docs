# gbps使用说明

## 概述

`gbps` 是 GBP（Global Buffer Page）的独立缓存服务端。启用 GBP 后，数据库实例在正常运行时会把可用于恢复的页面镜像发送到 `gbps`；当节点故障、实例重启恢复或集群恢复需要回放 redo 时，数据库会自动判断是否可以从 `gbps` 拉取页面来缩短恢复时间。

GBPS 的定位如下：

- 它是恢复加速组件，不是数据文件、日志文件或备份工具。
- 它不改变用户 SQL 的使用方式，启用后由数据库后台自动写入和读取。
- 如果 GBPS 不可用或窗口不满足恢复条件，数据库会回退到常规 redo 恢复，不需要用户手动干预。
- 在 CMS 部署中，建议优先通过 CMS 管理 GBPS；`gbps_ctl` 主要用于本机辅助检查和故障定位。

## 适用场景

GBPS 主要用于降低 oGRAC 两节点集群的故障恢复时间。当前典型使用场景是：一个节点宕机后，由另一个存活节点进行恢复；如果恢复过程中满足 GBP 可用条件，数据库会自动从 GBPS 读取页面并减少 redo 回放量。

正常业务运行时，用户不需要手动调用 GBPS 读取页面。`gbps_ctl exists`、`gbps_ctl dump` 等命令主要用于确认缓存和排查问题。

## 安装后的文件

正常安装 oGRAC 后，GBPS 相关文件位于数据库安装目录下：

| 文件 | 默认位置 | 说明 |
| --- | --- | --- |
| `gbps` | `$OGDB_HOME/bin/gbps` | GBPS 服务端程序。 |
| `gbps_ctl` | `$OGDB_HOME/bin/gbps_ctl` | GBPS 本地辅助管理命令，主要用于本机检查和故障定位。 |
| `gbps_contrl.sh` | `$OGDB_HOME/bin/gbps_contrl.sh` | CMS 管理 GBPS 资源时调用的脚本。 |
| `gbps.conf` | `$OGDB_DATA/cfg/gbps.conf` 或 `$OGDB_HOME/cfg/gbps.conf` | GBPS 服务端配置文件。 |

`gbps_ctl` 默认优先读取 `$OGDB_DATA/cfg/gbps.conf`，若该文件不存在，则读取 `$OGDB_HOME/cfg/gbps.conf`。也可以通过 `-c` 指定配置文件。

## 正常部署流程

### 1. 配置gbps.conf

每个部署 GBPS 的节点都需要配置 `gbps.conf`。以两节点对端部署为例，假设 node0 的主机 IP 为 `ip1`，node1 的主机 IP 为 `ip2`，两个节点都部署并启动 `gbps`。

如果 GBPS 监听所有网卡，可使用如下配置：

```ini
HOST=0.0.0.0
PORT=2611

ADMIN_HOST=127.0.0.1
ADMIN_PORT=2711

LOG_FILE=$OGDB_HOME/../../log/gbps/gbps.log
PID_FILE=$OGDB_HOME/run/gbps.pid

MAX_CACHE_PAGES=0
CAPACITY_EVICT_ON_WRITE=false
READ_END_MODE=async
READ_PHASE_TIMEOUT=3

VERBOSE=false
TIMING_DIAG=false
```

如果只希望监听本机业务 IP，可分别配置：

```ini
# node0 上的 gbps.conf
HOST=ip1
PORT=2611

# node1 上的 gbps.conf
HOST=ip2
PORT=2611
```

配置项说明：

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| `HOST` | `0.0.0.0` | GBPS 业务监听地址。数据库实例通过该地址写入和读取页面。 |
| `PORT` | `2611` | GBPS 业务监听端口，需要与数据库参数 `GBP_PORT` 一致。 |
| `ADMIN_HOST` | `127.0.0.1` | 管理端口监听地址，`gbps_ctl stats`、`gbps_ctl window` 等命令使用该地址。 |
| `ADMIN_PORT` | `2711` | 管理端口。 |
| `LOG_FILE` | `$OGDB_HOME/../../log/gbps/gbps.log` | GBPS 日志文件。 |
| `PID_FILE` | `$OGDB_HOME/run/gbps.pid` | GBPS 进程 PID 文件。 |
| `MAX_CACHE_PAGES` | `0` | 缓存页数量上限。`0` 表示不限制。 |
| `CAPACITY_EVICT_ON_WRITE` | `false` | 缓存达到上限后的行为。`false` 表示达到上限后拒绝新页；`true` 表示允许继续写入并后台淘汰旧页。 |
| `READ_END_MODE` | `async` | 恢复读阶段结束方式，可取 `async` 或 `sync`。一般保持默认值。 |
| `READ_PHASE_TIMEOUT` | `3` | 恢复读阶段空闲超时时间，单位为秒。`0` 表示不自动释放。 |
| `VERBOSE` | `false` | 是否输出详细日志。 |
| `TIMING_DIAG` | `false` | 是否输出耗时诊断日志。 |

### 2. 配置数据库侧USE_GBP

GBP 默认关闭。需要在每个数据库节点的 `ogracd.ini` 中启用。

以两节点对端部署为例，node0 的 IP 为 `ip1`，node1 的 IP 为 `ip2`。对端部署表示 node0 使用部署在 node1 上的 GBPS，node1 使用部署在 node0 上的 GBPS。因此 `GBP_IP` 按节点 ID 顺序配置为 `ip2,ip1`：第一个地址是 node0 对应的 GBPS 地址，第二个地址是 node1 对应的 GBPS 地址。

node0 的 `ogracd.ini` 示例：

```ini
USE_GBP=TRUE
GBP_FOR_RECOVERY=TRUE
GBP_IP=ip2,ip1
GBP_PORT=2611
LOCAL_GBP_HOST=ip1
```

node1 的 `ogracd.ini` 示例：

```ini
USE_GBP=TRUE
GBP_FOR_RECOVERY=TRUE
GBP_IP=ip2,ip1
GBP_PORT=2611
LOCAL_GBP_HOST=ip2
```

常用参数说明：

| 参数 | 默认值 | 生效方式 | 说明 |
| --- | --- | --- | --- |
| `USE_GBP` | `FALSE` | 重启生效 | GBP 总开关。设置为 `TRUE` 后，数据库实例才会连接并使用 GBPS。 |
| `GBP_FOR_RECOVERY` | `TRUE` | 重启生效 | 是否允许恢复阶段使用 GBP 加速。 |
| `GBP_IP` | `127.0.0.1` | 重启生效 | GBPS 服务地址列表。两节点对端部署时，若 node0 为 `ip1`、node1 为 `ip2`，配置为 `ip2,ip1`。 |
| `GBP_PORT` | `2611` | 重启生效 | GBPS 业务端口，需要与 `gbps.conf` 中的 `PORT` 一致。 |
| `LOCAL_GBP_HOST` | `127.0.0.1` | 重启生效 | 本节点连接 GBPS 时使用的本地地址。node0 配置为 `ip1`，node1 配置为 `ip2`。 |
| `GBP_RT_ANALYSIS` | `FALSE` | 重启生效 | 是否启用 GBP 运行时分析能力。普通使用场景保持默认值。 |
| `GBP_RT_PARSE_WORKERS` | `2` | 重启生效 | GBP 运行时分析解析线程数，取值范围 `[1,8]`。 |
| `GBP_RT_PAGE_OWNER_WORKERS` | `4` | 重启生效 | GBP 运行时页面归属处理线程数，取值范围 `[1,8]`。 |
| `GBP_ASSEMBLE_MAX_SCAN` | `300` | 立即生效 | 单次组装待发送页面时的最大扫描数量，取值范围 `[100,1000000]`。 |

### 3. 重启数据库并通过CMS启动GBPS

由于 `USE_GBP`、`GBP_IP`、`GBP_PORT` 等参数需要重启生效，修改 `ogracd.ini` 后需要重启数据库实例。重启后，数据库会自动连接 GBPS。用户不需要手动触发写页或读页。

在 CMS 部署中，GBPS 推荐作为 CMS 资源管理。启用 `USE_GBP=TRUE` 后，使用 CMS 启动和检查 GBPS：

```bash
su -s /bin/bash ograc
source ~/.bashrc

cms res -list
cms res -start gbps
cms stat -res gbps
```

如果需要停止 GBPS 资源，执行：

```bash
cms res -stop gbps
```

如果 `USE_GBP=FALSE`，CMS 会认为 `gbps` 资源被禁用：

- `cms res -list` 和 `cms stat` 会隐藏或跳过 `gbps`。
- `cms res -start gbps` 会失败或提示 `gbps disabled by USE_GBP=FALSE`。
- CMS 检查和自动拉起逻辑不会处理 `gbps`。

`cms res -list` 中与 `gbps` 相关的字段说明：

| 字段 | 说明 |
| --- | --- |
| `RESOURCE_NAME` | 资源名称，GBPS 资源为 `gbps`。 |
| `RESOURCE_TYPE` | 资源类型，GBPS 资源为 `gbps`。 |
| `RESOURCE_GROUP_NAME` | 资源所属资源组。 |
| `START_TIMEOUT(ms)` | CMS 启动该资源的超时时间，单位为毫秒。 |
| `STOP_TIMEOUT(ms)` | CMS 停止该资源的超时时间，单位为毫秒。 |
| `CHECK_TIMEOUT(ms)` | CMS 检查该资源状态的超时时间，单位为毫秒。 |
| `CHECK_INTERVAL(ms)` | CMS 周期性检查该资源的间隔，单位为毫秒。 |
| `HB_TIMEOUT(ms)` | 资源心跳超时时间，单位为毫秒。 |
| `RESTART_TIMES` | CMS 自动重启该资源的次数上限。 |
| `RESTART_INTERVAL` | CMS 自动重启该资源的间隔。 |
| `SCRIPT` | CMS 管理该资源时调用的脚本，GBPS 通常为 `gbps_contrl.sh`。 |

`cms stat -res gbps` 输出字段说明：

| 字段 | 说明 |
| --- | --- |
| `NODE_ID` | 节点 ID。 |
| `RESOURCE_NAME` | 资源名称，GBPS 资源为 `gbps`。 |
| `STAT` | 当前资源状态。`ONLINE` 表示在线，`OFFLINE` 表示离线，`UNKNOWN` 表示状态未知。 |
| `PRE_STAT` | 上一次资源状态。 |
| `TARGET_STAT` | CMS 期望资源达到的目标状态。 |
| `WORK_STAT` | CMS 内部工作状态。 |
| `LAST_CHECK` | 最近一次资源检查时间。 |
| `STAT_CHANGE` | 最近一次资源状态变化时间。 |

### 4. 使用gbps_ctl辅助检查

`gbps_ctl` 是本机辅助命令，适合在 CMS 状态异常、需要查看管理端口统计、查看恢复窗口或排查残留 READ_PHASE 时使用。正常启停优先使用 CMS。

示例：

```bash
gbps_ctl status
gbps_ctl stats
gbps_ctl window
```

## 如何确认组件正常

启用后建议按以下顺序检查：

```bash
# 1. 检查 CMS 是否登记并管理 gbps 资源
cms res -list

# 2. 检查 CMS 视角下的 gbps 资源状态
cms stat -res gbps

# 3. 如需进一步本机排查，再使用 gbps_ctl
gbps_ctl status
gbps_ctl stats

# 4. 如需确认恢复窗口，再查看 window
gbps_ctl window
```

通常情况下：

- `cms stat -res gbps` 中 `STAT` 显示 `ONLINE`，说明 CMS 认为 GBPS 资源在线。
- `gbps_ctl status` 显示 `gbps is running`，说明本机 GBPS 进程已启动。
- `gbps_ctl stats` 能返回 `OK`，说明本机 GBPS 管理端口可访问。
- 数据库运行一段时间后，`cache_total` 或 `cache_pages` 可能大于 0，说明已有页面进入 GBPS。
- `gbps_ctl window` 的 `begin`、`rcy`、`lrp` 是否可用于恢复由数据库自动判断，用户无需手工计算。

## gbps_ctl辅助命令参考

`gbps_ctl` 不作为 CMS 部署下的首选启停入口。用户日常管理应优先使用 `cms res -start gbps`、`cms res -stop gbps` 和 `cms stat -res gbps`；当需要在本机确认进程、管理端口、缓存统计或恢复窗口时，再使用本节命令。

### 语法

```bash
gbps_ctl {start|stop|stop_force|status|restart|stats|window|read_phase|force_read_end|exists|dump|query} [-c gbps.conf] [arg]
```

示例：

```bash
gbps_ctl status
gbps_ctl stats
gbps_ctl window
gbps_ctl status -c /path/to/gbps.conf
```

### start

本地启动 GBPS。CMS 部署下不建议优先使用该命令启停资源，日常启停应使用 `cms res -start gbps`。

成功启动时输出示例：

```text
gbps started: pid=12345 listen=0.0.0.0:2611 admin=127.0.0.1:2711 conf=/data/ograc/cfg/gbps.conf log=/data/ograc/log/gbps/gbps.log
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `pid` | GBPS 进程 ID。 |
| `listen` | GBPS 业务监听地址和端口，数据库实例通过该地址访问 GBPS。 |
| `admin` | GBPS 管理监听地址和端口，`gbps_ctl stats/window/read_phase` 等命令使用该地址。 |
| `conf` | 本次启动使用的配置文件。 |
| `log` | GBPS 日志文件。 |

如果服务已运行，输出示例：

```text
gbps is already running: pid=12345 conf=/data/ograc/cfg/gbps.conf
```

### status

查询 GBPS 进程状态。

运行中输出示例：

```text
gbps is running: pid=12345 listen=0.0.0.0:2611 admin=127.0.0.1:2711 conf=/data/ograc/cfg/gbps.conf log=/data/ograc/log/gbps/gbps.log
```

字段含义与 `start` 输出相同。

未运行输出示例：

```text
gbps is not running
```

如果存在过期 PID 文件，输出示例：

```text
gbps is not running; stale pid file=/data/ograc/server/run/gbps.pid
```

其中 `stale pid file` 表示 PID 文件存在，但对应进程已经不存在。

### stop

本地正常停止 GBPS。CMS 部署下不建议优先使用该命令停止资源，日常停止应使用 `cms res -stop gbps`。

输出示例：

```text
gbps stopped: pid=12345
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `pid` | 被停止的 GBPS 进程 ID。 |

如果服务本来未运行，输出：

```text
gbps is not running
```

### stop_force

本地强制停止 GBPS。该命令会先尝试正常停止，若短时间内未退出则强制结束进程。仅建议在 CMS 无法正常停止资源或本机排障时使用。

输出示例：

```text
gbps force stopped: pid=12345
```

字段说明与 `stop` 相同。

### restart

本地重启 GBPS。该命令等价于先执行 `stop`，再执行 `start`。CMS 部署下日常重启建议通过 CMS 停止后再启动资源。

### stats

查询 GBPS 简要统计。

输出示例：

```text
OK cache_total=100 pending_total=0 max_cache_pages=0 capacity_evict_on_write=0
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `OK` | 管理查询执行成功。 |
| `cache_total` | 当前 GBPS 缓存中的页面总数。 |
| `pending_total` | 当前 READ_PHASE 中待返回的页面总数。正常非恢复阶段通常为 `0`。 |
| `max_cache_pages` | 配置的缓存页上限。`0` 表示不限制。 |
| `capacity_evict_on_write` | 达到缓存上限时是否允许边写边淘汰。`0` 表示否，`1` 表示是。 |

### window

查询当前 GBP 恢复窗口。

输出示例：

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
queue_resets=1 max_reset_qid=0[rst=0 asn=1 blk=90 lfn=900 lsn=122900000] reset_lfns=0:900
queue_frontiers=8 min_frontier_qid=0[rst=0 asn=1 blk=200 lfn=2000 lsn=123400000] frontier_lfns=0:2000,1:2010
WINDOW_END
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `WINDOW_BEGIN`/`WINDOW_END` | 窗口信息开始和结束标记。 |
| `cache_pages` | 当前缓存页数。 |
| `pending_pages` | 当前 READ_PHASE 中等待返回的页面数。 |
| `max_cache_pages` | 缓存页上限。 |
| `capacity_evict_on_write` | 是否允许达到上限后继续写入并淘汰。`0` 表示否，`1` 表示是。 |
| `max_lsn` | GBPS 当前缓存页中最大的页面 LSN。 |
| `begin` | 当前窗口起点。恢复点早于或不在窗口内时，数据库不会使用该窗口加速。 |
| `rcy` | 当前窗口可恢复点。满足条件时，数据库可使用 GBPS 页面减少到该点之前的 redo 回放。 |
| `lrp` | 当前窗口覆盖的日志回放上界。 |
| `rst` | redo reset/incarnation 标识。 |
| `asn` | redo 日志文件序号。 |
| `blk` | redo 日志块号。 |
| `lfn` | redo 逻辑文件号。 |
| `lsn` | redo 逻辑序列号。 |
| `evict_in_progress` | 是否正在执行缓存淘汰。`0` 表示否，`1` 表示是。 |
| `purge_stable` | 清理后的窗口是否稳定。`1` 表示稳定。 |
| `empty_reason` | 窗口为空时的原因。为空表示当前没有额外说明。 |
| `queue_resets` | 有 reset 记录的队列数量。该字段可能不存在，表示当前没有 reset 记录。 |
| `max_reset_qid` | reset 点最大的队列 ID。 |
| `reset_lfns` | 各队列 reset 对应的 LFN 列表，格式为 `队列ID:LFN`。 |
| `queue_frontiers` | 有 frontier 记录的队列数量；显示为 `empty` 表示当前没有有效 frontier。 |
| `min_frontier_qid` | frontier 最小的队列 ID。 |
| `frontier_lfns` | 各队列 frontier 对应的 LFN 列表，格式为 `队列ID:LFN`。 |
| `missing` | 未上报 frontier 的队列 ID 列表。 |

### read_phase

查询恢复读阶段状态。

输出示例：

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

字段说明：

| 字段 | 说明 |
| --- | --- |
| `READ_PHASE_BEGIN`/`READ_PHASE_END` | READ_PHASE 信息开始和结束标记。 |
| `active` | 是否处于恢复读阶段。`0` 表示否，`1` 表示是。 |
| `ending` | READ_PHASE 是否正在结束。`0` 表示否，`1` 表示是。 |
| `elapsed_ms` | 本次 READ_PHASE 已持续时间，单位为毫秒。 |
| `idle_ms` | READ_PHASE 空闲时间，单位为毫秒。 |
| `inflight_reads` | 当前正在处理的读请求数量。 |
| `timeout_s` | READ_PHASE 自动释放超时时间，单位为秒。 |
| `dropped_page_writes` | READ_PHASE 期间被丢弃的写页请求数量。 |
| `timeout_warned` | 是否已经打印过超时告警。`0` 表示否，`1` 表示是。 |

### force_read_end

强制释放残留 READ_PHASE。仅在确认恢复读阶段异常残留时使用。

输出示例：

```text
OK active_before=1 ending_before=0 cleared=1 elapsed_ms=5000 dropped_page_writes=10 detached_pages=20
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `OK` | 命令执行成功。 |
| `active_before` | 执行命令前 READ_PHASE 是否处于 active 状态。 |
| `ending_before` | 执行命令前 READ_PHASE 是否正在结束。 |
| `cleared` | 是否完成清理。`1` 表示已清理。 |
| `elapsed_ms` | 被清理的 READ_PHASE 持续时间，单位为毫秒。 |
| `dropped_page_writes` | READ_PHASE 期间被丢弃的写页请求数量。 |
| `detached_pages` | 清理时释放的页面数量。 |

### exists

查询指定页面是否在 GBPS 缓存中。页面格式为 `<file>-<page>`。

输出示例：

```text
FOUND file=1 page=100 qid=4 pending=0 lsn=123456789 pcn=10 checksum=0xabcd trunc_lfn=1000 lrp_lfn=2000 writer_inst=0 writer_seq=123456789 reset[rst=0 asn=1 blk=90 lfn=900 lsn=122900000] frontier[rst=0 asn=1 blk=200 lfn=2000 lsn=123400000] cache_total=20
```

未找到时输出示例：

```text
NOT_FOUND file=1 page=100 qid=4 pending=0 reset[rst=0 asn=1 blk=90 lfn=900 lsn=122900000] frontier[rst=0 asn=1 blk=200 lfn=2000 lsn=123400000] cache_total=20
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `FOUND`/`NOT_FOUND` | 指定页面是否在缓存中。 |
| `file` | 数据文件编号。 |
| `page` | 页面编号。 |
| `qid` | 页面所属的 GBPS 内部队列 ID。 |
| `pending` | 该页面是否处于 READ_PHASE 待返回集合中。 |
| `lsn` | 页面头中的 LSN。仅 `FOUND` 时显示。 |
| `pcn` | 页面 PCN。仅 `FOUND` 时显示。 |
| `checksum` | 页面校验和。仅 `FOUND` 时显示。 |
| `trunc_lfn` | 该页面对应的窗口起点 LFN。仅 `FOUND` 时显示。 |
| `lrp_lfn` | 该页面对应的日志回放上界 LFN。仅 `FOUND` 时显示。 |
| `writer_inst` | 写入该页面的实例编号。仅 `FOUND` 时显示。 |
| `writer_seq` | 写入该页面时记录的写入序列。仅 `FOUND` 时显示。 |
| `reset[...]` | 该队列当前 reset 点。 |
| `frontier[...]` | 该队列当前 frontier 点。 |
| `cache_total` | 该队列当前缓存页数。 |

### dump

输出指定页面的详细诊断信息。该命令主要用于问题定位，普通巡检不需要执行。

输出字段说明：

| 字段 | 说明 |
| --- | --- |
| `DUMP_BEGIN`/`DUMP_END` | 诊断信息开始和结束标记。 |
| `file` | 数据文件编号。 |
| `page` | 页面编号。 |
| `qid` | 页面所属的 GBPS 内部队列 ID。 |
| `pending` | 是否处于 READ_PHASE 待返回集合中。 |
| `cache_total` | 该队列当前缓存页数。 |
| `writer_inst` | 写入该页面的实例编号。 |
| `writer_seq` | 写入该页面时记录的写入序列。 |
| `page_lsn` | 页面头中的 LSN。 |
| `page_pcn` | 页面 PCN。 |
| `page_checksum` | 页面校验和。 |
| `trunc_rst`/`trunc_asn`/`trunc_blk`/`trunc_lfn`/`trunc_lsn` | 页面窗口起点对应的 redo 坐标。 |
| `lrp_rst`/`lrp_asn`/`lrp_blk`/`lrp_lfn`/`lrp_lsn` | 页面日志回放上界对应的 redo 坐标。 |
| `reset` | 该队列当前 reset 点。 |
| `frontier` | 该队列当前 frontier 点。 |
| `trunc_hex` | 窗口起点坐标的十六进制内容。 |
| `lrp_hex` | 日志回放上界坐标的十六进制内容。 |
| `page_header_hex` | 页面头部十六进制预览。 |
| `page_tail_hex` | 页面尾部十六进制预览。 |

如果页面不存在，`dump` 输出与 `exists` 的 `NOT_FOUND` 输出一致。

### query

直接向管理端口发送查询命令。常用命令包括：

```bash
gbps_ctl query STATS
gbps_ctl query WINDOW
gbps_ctl query READ_PHASE
gbps_ctl query FORCE_READ_END
```

输出字段分别与 `stats`、`window`、`read_phase`、`force_read_end` 相同。

## 常见问题处理

### gbps_ctl提示找不到配置文件

确认已加载环境变量，或通过 `-c` 指定配置文件：

```bash
source ~/.bashrc
gbps_ctl status -c $OGDB_DATA/cfg/gbps.conf
```

### CMS无法启动gbps资源

检查 `ogracd.ini` 中是否已启用：

```ini
USE_GBP=TRUE
```

修改后重启数据库实例，再执行：

```bash
cms res -start gbps
cms stat -res gbps
```

### stats或window无法返回

优先检查：

- `cms stat -res gbps` 是否显示资源为 `ONLINE`。
- 如 CMS 显示异常，再使用 `gbps_ctl status` 确认本机进程是否显示 `gbps is running`。
- `gbps.conf` 中 `ADMIN_HOST` 和 `ADMIN_PORT` 是否正确。
- 当前用户是否已加载 `$OGDB_HOME`、`$OGDB_DATA` 等环境变量。

### READ_PHASE长时间未释放

先查询：

```bash
gbps_ctl read_phase
```

如果确认恢复读阶段已经异常残留，可执行：

```bash
gbps_ctl force_read_end
```

### 缓存页达到上限

如果设置了 `MAX_CACHE_PAGES` 且 `CAPACITY_EVICT_ON_WRITE=false`，缓存满后新页面会被拒绝写入。可根据环境容量调大 `MAX_CACHE_PAGES`，或设置：

```ini
CAPACITY_EVICT_ON_WRITE=true
```

调整 `gbps.conf` 后需要重启 GBPS。
