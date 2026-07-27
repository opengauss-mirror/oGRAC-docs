# RBP加速恢复参考

## 概述

RBP（Recovery Buffer Pool，恢复缓冲池）是一种面向恢复加速的页面缓存能力。启用 RBP 后，数据库在运行期间将恢复可能需要的部分页面和恢复窗口信息写入 RBPS，并在实例或节点故障恢复时复用 RBPS 中保存的页面，从而减少仅依赖 Redo 回放重建页面的工作量。

RBPS（RBP Server）是 RBP 的页面缓存服务端，用于接收、保存和提供恢复阶段所需的页面。恢复过程中，数据库会根据 RBPS 中的恢复窗口和安全检查结果决定是否使用 RBP：满足条件时可缩短节点故障恢复等场景下的恢复时间，帮助满足更短的恢复时间目标（RTO）；如果 RBPS 不可用、恢复窗口不满足条件或安全检查未通过，数据库会自动回退到常规恢复。本文档说明数据库侧如何配置 RBP 加速恢复。

RBPS 服务端配置、CMS 管理方式和 `rbps_ctl` 命令请参见[rbps使用说明](../tool_and_commandreference/server_tool/rbps_instructions.md)。RBP 相关 GUC 参数的含义、取值范围、默认值和生效方式请参见[数据库参考](./database_reference.md)中的 RBP 参数说明。

## 性能影响

启用 RBP 后，数据库运行期间需要额外维护恢复加速所需状态，并将部分页面写入 RBPS；开启 `RBP_RT_ANALYSIS` 后，还会持续解析日志、维护页面归属等运行期分析信息。这些工作会消耗 CPU、内存、I/O 和网络资源，并可能与业务事务争用缓冲池、日志处理和网络带宽，因此会对业务性能带来一定下降。生产环境建议结合实际业务负载评估该能力，并将 RBP/RBPS 流量与业务网络分离。

## 配置前提

- 已在需要承载页面缓存的节点部署 RBPS，并确认 RBPS 监听地址和端口。
- 已确认数据库节点编号、本机 RBP 通信地址，以及数据库节点到 RBPS 监听端口的网络连通性。
- RBP 通信可以与业务访问使用不同网卡。生产环境推荐为 RBP/RBPS 流量配置独立网卡或独立网络，降低恢复加速流量对业务网络的影响。
- 两节点 DTC 部署中，数据库侧通常配置为连接对端 RBPS。请根据实际部署关系确认每个数据库节点需要连接的 RBPS 地址。
- 修改基础 RTO 或 RBP 重启类参数后，需要重启数据库实例。

## 推荐配置

### 恢复加速配置

两节点 DTC 场景中，数据库侧通常按部署规划配置 RBPS 路由目标地址。如下示例中，RBP 使用独立网络地址：

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

其中，`node1_rbp_ip` 表示 `node1` 的 RBPS IPv4 地址，`node0_rbp_ip` 表示 `node0` 的 RBPS IPv4 地址；配置时需替换为实际 IPv4 地址，并按节点编号与 RBPS 路由关系填写地址顺序。`RBP_IP` 最多可配置 8 个 IPv4 地址，多个地址使用英文逗号分隔，地址列表不支持空项或分号分隔。推荐将 RBP/RBPS 流量与业务流量分离，避免页面写入、读取和恢复阶段的网络流量影响业务访问。


## 数据库参数配置

可以通过数据库配置文件或 `ALTER SYSTEM` 修改参数。使用 `ALTER SYSTEM` 时，需要在每个数据库节点分别执行。

### 基础RTO参数配置

除 RBP 参数外，恢复加速场景还建议同步调整以下数据库基础 RTO 参数：

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

### RBP参数配置

RBP 重启类参数建议写入参数文件后重启实例：

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

配置完成后重启数据库实例，并确认 RBPS 已启动。

## 配置检查

数据库侧可先查看参数是否已经加载：

```sql
SHOW PARAMETER USE_RBP;
SHOW PARAMETER RBP_IP;
SHOW PARAMETER RBP_PORT;
```

随后检查 RBPS 服务端状态和恢复窗口：

```bash
cms stat -res rbps
rbps_ctl stats
rbps_ctl window
```

如果数据库日志中可以看到 RBP 连接或心跳相关信息，说明数据库侧与 RBPS 基础链路已经建立。恢复阶段是否实际使用 RBP 仍由数据库根据恢复窗口和安全检查结果自动判断。

## 常见配置问题

### 数据库重启后未连接到RBPS

- 确认重启类参数已经写入参数文件并重启实例。
- 确认数据库节点可以访问 `RBP_IP` 中配置的 RBPS 监听端口。
- 确认 `RBP_PORT` 与 `rbps.conf` 中的 `PORT` 一致。

### 恢复阶段没有使用RBP

- 确认 `USE_RBP` 和 `RBP_FOR_RECOVERY` 已启用并在重启后生效。
- 确认 `RBP_IP` 已按实际对端关系配置。
- 确认 `rbps_ctl window` 中存在可用于恢复的窗口。
- 查看数据库日志中是否存在连接失败、地址配置不匹配或回退到常规恢复等信息。

即使数据库侧配置正确，如果当前恢复窗口不满足使用条件，数据库仍会自动回退到常规恢复。

### 运行期分析未启动

- 确认 `RBP_RT_ANALYSIS` 已设置为 `TRUE` 并重启实例。
- 确认当前环境属于支持运行期分析的恢复加速场景。
- 如果资源占用较高，可保持 `RBP_RT_ANALYSIS=FALSE`，仅使用基础 RBP 恢复加速路径。
