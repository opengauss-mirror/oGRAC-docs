# rbps使用说明

## 概述

`rbps` 是 RBP 的页面缓存服务端。启用 RBP 后，数据库可在恢复过程中使用 RBPS 中保存的页面，从而缩短节点故障恢复场景下的恢复时间。

RBPS 的定位如下：

- 恢复加速组件，不是数据文件、日志文件或备份工具。
- 不改变用户 SQL 的使用方式，启用后由数据库自动完成页面写入和读取。
- 如果 RBPS 不可用、RBP 窗口不满足恢复条件，或恢复安全检查未通过，数据库会回退到常规恢复。
- 在 CMS 部署中，建议优先通过 CMS 管理 `rbps` 资源；`rbps_ctl` 主要用于本机辅助检查和故障定位。


## 安装配置

正常安装 oGRAC 后，RBPS 相关文件位于数据库安装目录下：

| 文件 | 默认位置 | 说明 |
| --- | --- | --- |
| `rbps` | `$OGDB_HOME/bin/rbps` | RBPS 服务端程序。 |
| `rbps_ctl` | `$OGDB_HOME/bin/rbps_ctl` | RBPS 本地辅助管理命令，用于启停、状态检查和管理端口查询。 |
| `rbps_contrl.sh` | `$OGDB_HOME/bin/rbps_contrl.sh` | CMS 管理 `rbps` 资源时调用的脚本。 |
| `rbps.conf` | `$OGDB_DATA/cfg/rbps.conf` | RBPS 服务端配置文件。 |

`rbps_ctl` 默认优先读取 `$OGDB_DATA/cfg/rbps.conf`，也可以通过 `-c` 指定配置文件。

## 配置rbps.conf

每个部署 RBPS 的节点都需要配置 `rbps.conf`。常用配置如下：

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

配置项说明：

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| `HOST` | `0.0.0.0` | RBPS 监听地址。数据库实例通过该地址写入和读取页面。 |
| `PORT` | `2611` | RBPS 监听端口，需要与数据库侧连接端口配置一致。 |
| `ADMIN_HOST` | `127.0.0.1` | 管理端口监听地址，`rbps_ctl stats`、`rbps_ctl window` 等命令使用该地址。 |
| `ADMIN_PORT` | `2711` | 管理端口。设置为不可用端口会导致管理查询失败。 |
| `LOG_FILE` | `$OGDB_HOME/../../log/rbps/rbps.log` | RBPS 日志文件。 |
| `PID_FILE` | `$OGDB_HOME/run/rbps.pid` | RBPS 进程 PID 文件。 |
| `MAX_CACHE_PAGES` | `3000000` | 缓存页数量上限。`0` 表示不限制。 |
| `CAPACITY_EVICT_ON_WRITE` | `false` | 缓存达到上限后的行为。`false` 表示达到上限后拒绝新页面，`true` 表示允许继续写入并由后台淘汰旧页面。 |
| `READ_END_MODE` | `async` | 恢复读阶段结束方式，可取 `async` 或 `sync`。一般保持默认值。 |
| `READ_PHASE_TIMEOUT` | `3` | orphan READ_PHASE 自动释放的空闲超时时间，单位为秒。`0` 表示关闭自动释放。 |
| `VERBOSE` | `false` | 是否输出更详细日志。 |
| `TIMING_DIAG` | `false` | 是否输出耗时诊断日志。 |

## 正常部署流程

### 1. 准备RBPS服务端配置

在每个 RBPS 节点确认 `rbps.conf` 中 `HOST`、`PORT`、`LOG_FILE` 正确。如果使用 CMS 管理，确认 `$OGDB_HOME/bin/rbps_contrl.sh` 和 `$OGDB_HOME/bin/rbps_ctl` 可执行。

### 2. 确认数据库侧配置

在每个数据库节点按[RBP加速恢复参考](../../database_reference/rbp_accelerated_recovery_reference.md)完成数据库侧配置，并确保数据库侧连接地址、端口与 RBPS 服务端监听配置匹配。此处不展开数据库 GUC 参数说明。

### 3. 通过CMS启动RBPS

CMS 部署中，安装流程会尝试预注册 `rbps` 资源。数据库侧启用 RBP 后，可使用 CMS 启动和检查：

```bash
su -s /bin/bash ograc
source ~/.bashrc

cms res -list
cms res -start rbps
cms stat -res rbps
```

停止 RBPS：

```bash
cms res -stop rbps
```

如果数据库侧关闭 RBP，CMS 会认为 `rbps` 资源被禁用：

- `cms res -list` 和 `cms stat` 会隐藏 `rbps`。
- `cms res -start rbps` 会失败并提示资源被禁用。

### 4. 重启数据库实例

按数据库侧配置参考修改重启类参数后，需要重启数据库实例。实例启动后，数据库会加载 RBP 配置并连接 RBPS。正常情况下日志中可以看到 RBP 连接或心跳相关信息。

## 如何确认组件正常

建议按如下顺序检查：

```bash
# 1. CMS 是否登记并管理 rbps 资源
cms res -list

# 2. CMS 视角下 rbps 是否在线，`STAT` 显示 `ONLINE`，说明 CMS 认为 RBPS 资源在线。
cms stat -res rbps 

# 3. 本机进程和管理端口是否正常
rbps_ctl status     # 显示 `rbps is running`，说明本机 RBPS 进程已启动。
rbps_ctl stats      # 返回 `OK`，说明本机 RBPS 管理端口可访问。
```

## rbps_ctl辅助命令参考

`rbps_ctl` 不作为 CMS 部署下的首选启停入口。日常管理应优先使用 `cms res -start rbps`、`cms res -stop rbps` 和 `cms stat -res rbps`；当需要在本机确认进程、管理端口、缓存统计或恢复窗口时，再使用本节命令。

###   start

本地启动 RBPS。CMS 部署下日常启动建议使用 `cms res -start rbps`。

成功启动时输出示例：

```text
rbps started: pid=12345 listen=0.0.0.0:2611 admin=127.0.0.1:2711 conf=/data/ograc/cfg/rbps.conf log=/data/ograc/log/rbps/rbps.log
```

如果服务已运行，输出示例：

```text
rbps is already running: pid=12345 conf=/data/ograc/cfg/rbps.conf
```

###  status

查询 RBPS 进程状态。

运行中输出示例：

```text
rbps is running: pid=12345 listen=0.0.0.0:2611 admin=127.0.0.1:2711 conf=/data/ograc/cfg/rbps.conf log=/data/ograc/log/rbps/rbps.log
```

未运行输出示例：

```text
rbps is not running
```


### stop、stop_force

`stop` 本地正常停止 RBPS。`stop_force` 会先尝试正常停止，短时间内未退出则强制结束进程。CMS 部署下日常停止建议使用 `cms res -stop rbps`，`stop_force` 仅建议在 CMS 无法正常停止资源或本机排障时使用。

输出示例：

```text
rbps stopped: pid=12345
rbps force stopped: pid=12345
```

### restart

本地重启 RBPS，等价于先执行 `stop`，再执行 `start`。

### stats

查询 RBPS 简要统计。

输出示例：

```text
OK cache_total=100 pending_total=0 max_cache_pages=0 capacity_evict_on_write=0
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `OK` | 管理查询执行成功。 |
| `cache_total` | 当前 RBPS 缓存中的页面总数。 |
| `pending_total` | 保留字段、当前恢复读阶段待返回的页面总数。 |
| `max_cache_pages` | 配置的缓存页上限，`0` 表示不限制。 |
| `capacity_evict_on_write` | 达到缓存上限时是否允许边写边淘汰，`0` 表示否，`1` 表示是。 |

### window

查询当前 RBP 恢复窗口。

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
WINDOW_END
```

常用字段说明：

| 字段 | 说明 |
| --- | --- |
| `cache_pages` | 当前缓存页数。 |
| `pending_pages` | 当前恢复读阶段等待返回的页面数。 |
| `max_lsn` | RBPS 当前缓存页中的最大页面 LSN。 |
| `begin` | 当前窗口起点。恢复点早于或不在窗口内时，数据库不会使用该窗口加速。 |
| `rcy` | 当前窗口可恢复点。满足条件时，数据库可使用 RBPS 页面缩短恢复时间。 |
| `lrp` | 当前窗口覆盖的日志回放上界。 |
| `evict_in_progress` | 是否正在执行缓存淘汰。 |
| `purge_stable` | 清理后的窗口是否稳定。 |
| `empty_reason` | 窗口为空时的原因。 |

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
| `active` | 是否处于恢复读阶段，`1` 表示是。 |
| `ending` | READ_PHASE 是否正在结束。 |
| `elapsed_ms` | 本次 READ_PHASE 已持续时间，单位为毫秒。 |
| `idle_ms` | READ_PHASE 空闲时间，单位为毫秒。 |
| `inflight_reads` | 当前正在处理的读请求数量。 |
| `timeout_s` | READ_PHASE 自动释放超时时间，单位为秒。 |
| `dropped_page_writes` | READ_PHASE 期间被丢弃的写页请求数量。 |
| `timeout_warned` | 是否已经打印过超时告警。 |

### force_read_end

强制释放残留 READ_PHASE。仅在确认恢复读阶段异常残留时使用。

输出示例：

```text
OK active_before=1 ending_before=0 cleared=1 elapsed_ms=5000 dropped_page_writes=10 detached_pages=20
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `active_before` | 执行命令前 READ_PHASE 是否处于 active 状态。 |
| `ending_before` | 执行命令前 READ_PHASE 是否正在结束。 |
| `cleared` | 是否完成清理，`1` 表示已清理。 |
| `elapsed_ms` | 被清理的 READ_PHASE 持续时间，单位为毫秒。 |
| `dropped_page_writes` | READ_PHASE 期间被丢弃的写页请求数量。 |
| `detached_pages` | 清理时释放的页面数量。 |

### exists

查询指定页面是否在 RBPS 缓存中。页面格式为 `<file>-<page>`，也可使用 `<file>_<page>` 或 `<file>/<page>`。

输出示例：

```text
FOUND file=1 page=100 lsn=123456789 pcn=10 cache_total=20
```

未找到时输出示例：

```text
NOT_FOUND file=1 page=100 cache_total=20
```

### dump

输出指定页面的详细诊断信息，主要用于问题定位。页面不存在时，输出与 `exists` 的 `NOT_FOUND` 一致。

输出示例：

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

常见字段包括：

| 字段 | 说明 |
| --- | --- |
| `DUMP_BEGIN`/`DUMP_END` | 诊断信息开始和结束标记。 |
| `file`/`page` | 数据文件编号和页面编号。 |
| `pending` | 是否处于BATCH_READ恢复读阶段待返回集合中。 |
| `writer_inst` | 写入该页面的实例编号。 |
| `page_lsn`/`page_pcn` | 页面头中的 LSN 和 PCN。 |
| `page_header_hex`/`page_tail_hex` | 页面头部和尾部十六进制预览。 |

### query

直接向管理端口发送查询命令。常用命令包括：

```bash
rbps_ctl query STATS
rbps_ctl query WINDOW
rbps_ctl query READ_PHASE
rbps_ctl query FORCE_READ_END
```

输出字段分别与 `stats`、`window`、`read_phase`、`force_read_end` 相同。

## 常见问题处理

### rbps_ctl提示找不到配置文件

确认已加载环境变量，或通过 `-c` 指定配置文件：

```bash
source ~/.bashrc
rbps_ctl status -c $OGDB_DATA/cfg/rbps.conf
```

### CMS无法启动rbps资源

先按[RBP加速恢复参考](../../database_reference/rbp_accelerated_recovery_reference.md)确认数据库侧已完成配置并重启实例，再执行：

```bash
cms res -start rbps
cms stat -res rbps
```

### stats或window无法返回

优先检查：

- `cms stat -res rbps` 是否显示资源为 `ONLINE`。
- 如 CMS 显示异常，再使用 `rbps_ctl status` 确认本机进程是否显示 `rbps is running`。
- `rbps.conf` 中 `ADMIN_HOST` 和 `ADMIN_PORT` 是否正确。
- 当前用户是否已加载 `$OGDB_HOME`、`$OGDB_DATA` 等环境变量。

### 恢复阶段没有使用RBP

按以下顺序检查：

- 数据库侧是否已按[RBP加速恢复参考](../../database_reference/rbp_accelerated_recovery_reference.md)完成配置并重启实例。
- RBPS 服务地址和端口是否与数据库侧连接配置匹配，且服务端口可从数据库节点访问。
- `rbps_ctl window` 是否有有效窗口。
- 数据库日志中是否存在连接失败、地址配置不匹配或回退到常规恢复等信息。

即使 RBP 已启用，如果当前恢复窗口不满足使用条件，数据库仍会自动回退到常规恢复。

### READ_PHASE长时间未释放

先查询：

```bash
rbps_ctl read_phase
```

如果确认恢复读阶段已经异常残留，可执行：

```bash
rbps_ctl force_read_end
```

### 缓存页达到上限

如果设置了 `MAX_CACHE_PAGES` 且 `CAPACITY_EVICT_ON_WRITE=false`，缓存满后新页面会被拒绝写入。可根据环境容量调大 `MAX_CACHE_PAGES`，或设置：

```ini
CAPACITY_EVICT_ON_WRITE=true
```

调整 `rbps.conf` 后需要重启 RBPS。
