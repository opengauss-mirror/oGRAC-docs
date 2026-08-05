# CMS 磁盘使用率监控与自动只读保护

## 1. 功能概述

CMS 磁盘使用率监控与自动只读保护功能用于在 oGRAC 资源池化集群中监控关键磁盘空间使用情况。当磁盘使用率达到配置阈值时，CMS 可以自动将数据库切换为 `readonly`，避免数据库继续写入导致磁盘耗尽、日志无法落盘、数据文件扩展失败或集群异常。

在资源池化部署场景中，数据库通常依赖共享存储和 DSS 管理数据盘、Redo 盘、归档盘等关键存储资源。磁盘空间耗尽可能影响数据库写入能力和集群稳定性。因此，CMS 会周期性采集以下对象的空间使用情况，并根据配置决定是否触发 `readonly` 或 `readwrite` 保护动作：

- 本地数据库数据目录所在文件系统。
- DSS VG。

### 主要能力

1. 周期性采集本地数据库数据目录所在文件系统的磁盘使用率。
2. 周期性采集 DSS VG 的磁盘使用率。
3. 通过 CMS 命令查看当前磁盘使用率、阈值、自动保护开关、冷却时间和只读保护状态。
4. 当磁盘使用率达到配置阈值时，自动将数据库切换为 `readonly`。
5. 当磁盘使用率恢复到阈值以下且满足冷却时间要求后，自动恢复数据库为 `readwrite`。
6. 通过命令动态修改阈值、巡检周期、冷却时间和自动保护开关。
7. 参数修改后写入 `cms.ini` 配置文件。

## 2. 工作机制

CMS 启动后会创建磁盘使用率巡检线程。该线程按照 `_DISK_USAGE_CHECK_INTERVAL` 配置的周期执行磁盘检查。

每轮检查主要执行以下流程：

1. 读取 `cms.ini` 中的磁盘保护相关配置。
2. 采集本地数据库目录所在文件系统的容量、已用空间、剩余空间和使用率。
3. 尝试发现 DSS 环境，并通过 `dsscmd` 采集 DSS VG 的容量、已用空间、剩余空间和使用率。
4. 生成磁盘使用率快照，供 `cms diskreadonly -show` 查询。
5. 判断是否存在采集成功且使用率达到阈值的磁盘对象。
6. 如果自动保护开关已开启且满足 `readonly` 触发条件，则向本节点数据库资源发送 `readmode switch` 消息，将数据库切换为 `readonly`。
7. 如果数据库已由该功能触发 `readonly`，且磁盘使用率恢复到阈值以下并满足冷却时间要求，则尝试自动恢复数据库为 `readwrite`。

### 自动 readonly 判断条件

```text
磁盘使用率 >= _DISK_USAGE_THRESHOLD
```

只有采集成功的磁盘对象会参与触发判断。采集失败的对象会在查询结果中显示为 `ERROR`，但不会作为成功告警对象触发 `readonly`。

## 3. 功能启用与关闭

### 3.1 查看当前功能状态

```bash
cms diskreadonly -show
```

示例输出：

```text
DISK_USAGE_CHECK_INTERVAL = 30
DISK_USAGE_THRESHOLD = 85
DISK_USAGE_PROTECT_ENABLE = TRUE
DISK_USAGE_READONLY_COOLDOWN = 120
DISK_USAGE_READONLY_STATE = NORMAL
TYPE   NAME   SOURCE                           TOTAL_GB   USED_GB   FREE_GB   USE%     THRESHOLD   STATUS   LAST_CHECK            INFO
LOCAL  local  /home/ogracdba/data/data         100.000    40.000    60.000    40.000   85          OK       2026-08-05 12:00:00  OK
DSS    data   /opt/ograc/dss/bin/dsscmd        2048.000   1200.000  848.000   58.594   85          OK       2026-08-05 12:00:00  discovered from environment
```

说明：

- `cms diskreadonly -show` 查询的是 CMS 内存中的最新磁盘使用率快照。
- 该命令不会直接读取 `cms.ini` 文件。
- 修改参数后，参数会写入 `cms.ini`，但 `-show` 的结果可能需要等待下一轮巡检后刷新。
- 默认巡检周期为 30 秒，因此修改参数后最多可能存在一个巡检周期的显示延迟。

### 3.2 开启自动只读保护

```bash
cms diskreadonly -auto_change True
```

执行成功后，`cms.ini` 中的配置会更新为：

```ini
_DISK_USAGE_PROTECT_ENABLE = TRUE
```

开启后，当磁盘使用率达到阈值时，CMS 会自动触发数据库切换为 `readonly`。

### 3.3 关闭自动只读保护

```bash
cms diskreadonly -auto_change False
```

执行成功后，`cms.ini` 中的配置会更新为：

```ini
_DISK_USAGE_PROTECT_ENABLE = FALSE
```

关闭后，CMS 仍会采集和展示磁盘使用率，但不会自动触发 `readonly`/`readwrite` 切换。

> 注意：关闭自动只读保护不会自动恢复数据库为 `readwrite`。如果数据库已经被切换为 `readonly`，需要确认磁盘空间和数据库状态满足恢复条件后手动执行恢复命令。

## 4. 常见操作

### 4.1 查看磁盘使用率和保护状态

```bash
cms diskreadonly -show
```

用于查看当前磁盘使用率、保护配置和只读保护状态。

### 4.2 设置磁盘使用率阈值

```bash
cms diskreadonly -set threshold 90
```

表示当任一成功采集的磁盘对象使用率达到 90% 时，触发 `readonly` 保护判断。

修改后，`cms.ini` 中的配置为：

```ini
_DISK_USAGE_THRESHOLD = 90
```

### 4.3 设置自动切换冷却时间

```bash
cms diskreadonly -set cooldown 300
```

表示两次 `readonly`/`readwrite` 动作之间至少间隔 300 秒。

修改后，`cms.ini` 中的配置为：

```ini
_DISK_USAGE_READONLY_COOLDOWN = 300
```

### 4.4 设置磁盘使用率巡检周期

```bash
cms diskreadonly -set interval 60
```

表示 CMS 每 60 秒采集一次磁盘使用率。

修改后，`cms.ini` 中的配置为：

```ini
_DISK_USAGE_CHECK_INTERVAL = 60
```

### 4.5 手动恢复数据库为 readwrite

```bash
cms diskreadonly -recover-now
```

该命令用于手动触发数据库恢复为 `readwrite`。

适用场景：

1. 数据库已经因磁盘保护被自动切换为 `readonly`。
2. 运维人员已经完成磁盘清理。
3. 已确认磁盘空间恢复到安全范围。
4. 已确认数据库可以恢复写入。

注意：

- 如果数据库当前已经是 `readwrite`，命令会返回数据库已处于 `readwrite` 或恢复成功类信息。
- 如果数据库资源未注册、CMS 与数据库通信失败，或数据库状态不满足恢复条件，则恢复会失败。
- 手动恢复 `readwrite` 会刷新最近一次动作时间，后续自动切换仍会受到 `cooldown` 限制。

## 5. 命令汇总

| 命令 | 功能说明 | 是否写入 `cms.ini` | 备注 |
| --- | --- | --- | --- |
| `cms diskreadonly -show` | 查看磁盘使用率和只读保护配置快照 | 否 | 查询 CMS 内存快照，不直接读取配置文件 |
| `cms diskreadonly -auto_change True` | 开启自动 `readonly`/`readwrite` 保护 | 是 | 写入 `_DISK_USAGE_PROTECT_ENABLE = TRUE` |
| `cms diskreadonly -auto_change False` | 关闭自动 `readonly`/`readwrite` 保护 | 是 | 写入 `_DISK_USAGE_PROTECT_ENABLE = FALSE` |
| `cms diskreadonly -set threshold <percent>` | 设置磁盘使用率保护阈值 | 是 | 写入 `_DISK_USAGE_THRESHOLD` |
| `cms diskreadonly -set cooldown <seconds>` | 设置 `readonly`/`readwrite` 动作冷却时间 | 是 | 写入 `_DISK_USAGE_READONLY_COOLDOWN` |
| `cms diskreadonly -set interval <seconds>` | 设置磁盘使用率巡检周期 | 是 | 写入 `_DISK_USAGE_CHECK_INTERVAL` |
| `cms diskreadonly -recover-now` | 手动恢复数据库为 `readwrite` | 否 | 用于磁盘清理后手动恢复写入 |

## 6. 查询结果字段说明

`cms diskreadonly -show` 会展示磁盘保护配置和磁盘使用率明细。

### 6.1 配置字段

| 字段 | 含义 | 示例 |
| --- | --- | --- |
| `DISK_USAGE_CHECK_INTERVAL` | 磁盘使用率巡检周期，单位为秒 | `30` |
| `DISK_USAGE_THRESHOLD` | 磁盘使用率保护阈值，单位为百分比 | `85` |
| `DISK_USAGE_PROTECT_ENABLE` | 是否开启自动 `readonly`/`readwrite` 保护 | `TRUE` |
| `DISK_USAGE_READONLY_COOLDOWN` | `readonly`/`readwrite` 动作冷却时间，单位为秒 | `120` |
| `DISK_USAGE_READONLY_STATE` | 当前自动只读保护状态 | `NORMAL` |

### 6.2 磁盘明细字段

| 字段 | 含义 |
| --- | --- |
| `TYPE` | 磁盘对象类型。`LOCAL` 表示本地数据库目录，`DSS` 表示 DSS VG |
| `NAME` | 磁盘对象名称 |
| `SOURCE` | 磁盘对象来源。本地盘显示数据目录路径，DSS 显示 `dsscmd` 路径 |
| `TOTAL_GB` | 总容量，单位为 GB |
| `USED_GB` | 已使用容量，单位为 GB |
| `FREE_GB` | 剩余容量，单位为 GB |
| `USE%` | 当前磁盘使用率 |
| `THRESHOLD` | 当前对象使用的保护阈值 |
| `STATUS` | 当前对象状态。`OK` 表示正常，`WARN` 表示达到阈值，`ERROR` 表示采集失败 |
| `LAST_CHECK` | 最近一次采集时间 |
| `INFO` | 采集说明或错误信息 |

## 7. 参数详细说明

### 7.1 `_DISK_USAGE_CHECK_INTERVAL`

| 项目 | 说明 |
| --- | --- |
| 参数含义 | CMS 磁盘使用率巡检周期 |
| 参数类型 | 整数 |
| 单位 | 秒 |
| 取值范围 | `5 ~ 3600` |
| 默认值 | `30` |
| 是否支持命令修改 | 支持 |
| 修改命令 | `cms diskreadonly -set interval <seconds>` |
| 是否写入 `cms.ini` | 是 |

示例：

```bash
cms diskreadonly -set interval 60
```

说明：

- 控制 CMS 多久采集一次磁盘使用率。
- 巡检对象包括本地数据库目录所在文件系统和 DSS VG。
- 值越小，磁盘水位变化感知越及时，但采集频率越高。
- 值越大，对系统影响越小，但磁盘水位变化发现会更慢。
- 修改后会写入 `cms.ini`。
- `cms diskreadonly -show` 查询的是 CMS 内存快照，修改后可能需要等待下一轮巡检才能显示新值。

### 7.2 `_DISK_USAGE_THRESHOLD`

| 项目 | 说明 |
| --- | --- |
| 参数含义 | 磁盘使用率保护阈值 |
| 参数类型 | 整数 |
| 单位 | 百分比 |
| 取值范围 | `1 ~ 100` |
| 默认值 | `85` |
| 是否支持命令修改 | 支持 |
| 修改命令 | `cms diskreadonly -set threshold <percent>` |
| 是否写入 `cms.ini` | 是 |

示例：

```bash
cms diskreadonly -set threshold 90
```

说明：

- 当任一成功采集的磁盘对象满足 `USE% >= _DISK_USAGE_THRESHOLD` 时，CMS 认为该对象达到保护条件。
- 本地数据库目录和 DSS VG 均使用该阈值判断。
- 只有采集成功的磁盘对象会参与 `readonly` 触发判断。
- 采集失败的磁盘对象会显示为 `ERROR`，但不会作为成功告警对象触发 `readonly`。
- 阈值过低可能导致数据库过早进入 `readonly`。
- 阈值过高可能导致磁盘空间风险发现过晚。
- 建议结合业务写入速度、Redo 产生速度、归档增长速度和运维响应时间设置。

### 7.3 `_DISK_USAGE_PROTECT_ENABLE`

| 项目 | 说明 |
| --- | --- |
| 参数含义 | 是否开启磁盘使用率自动 `readonly`/`readwrite` 保护 |
| 参数类型 | 布尔型 |
| 取值范围 | `TRUE`、`FALSE` |
| 默认值 | `TRUE` |
| 是否支持命令修改 | 支持 |
| 修改命令 | `cms diskreadonly -auto_change True/False` |
| 是否写入 `cms.ini` | 是 |

开启：

```bash
cms diskreadonly -auto_change True
```

关闭：

```bash
cms diskreadonly -auto_change False
```

说明：

- 设置为 `TRUE` 时，CMS 会在磁盘使用率达到阈值后自动触发 `readonly`。
- 设置为 `FALSE` 时，CMS 仍会采集磁盘使用率，但不会自动触发 `readonly`/`readwrite` 切换。
- 该参数只控制自动保护动作，不影响 `cms diskreadonly -show` 查询能力。
- 如果数据库已经处于 `readonly`，关闭该参数不会自动恢复 `readwrite`。
- 如需恢复 `readwrite`，应使用 `cms diskreadonly -recover-now`，或通过其他数据库管理方式恢复。

### 7.4 `_DISK_USAGE_READONLY_COOLDOWN`

| 项目 | 说明 |
| --- | --- |
| 参数含义 | `readonly`/`readwrite` 动作冷却时间 |
| 参数类型 | 整数 |
| 单位 | 秒 |
| 取值范围 | `1 ~ 3600` |
| 默认值 | `120` |
| 是否支持命令修改 | 支持 |
| 修改命令 | `cms diskreadonly -set cooldown <seconds>` |
| 是否写入 `cms.ini` | 是 |

示例：

```bash
cms diskreadonly -set cooldown 300
```

说明：

- 用于避免数据库在 `readonly` 和 `readwrite` 之间频繁切换。
- CMS 会记录最近一次 `readonly`/`readwrite` 动作时间。
- 未超过 `cooldown` 时，即使命中切换条件，也不会立即执行动作。
- 手动恢复 `readwrite` 会刷新最近一次动作时间。
- 冷却时间按照最近一次 `readonly`/`readwrite` 动作时间计算，而不是只按照第一次触发 `readonly` 的时间计算。
- `cooldown` 过短可能导致数据库读写状态频繁变化。
- `cooldown` 过长可能导致磁盘恢复后数据库长时间保持 `readonly`。

## 8. 只读保护状态

`DISK_USAGE_READONLY_STATE` 用于展示当前自动只读保护状态。

| 状态 | 含义 |
| --- | --- |
| `NORMAL` | 当前未触发 `readonly`，磁盘使用率未达到保护条件，或保护已恢复 |
| `READONLY_PENDING` | 已达到 `readonly` 条件，但仍处于冷却时间内，暂不执行 `readonly` |
| `READONLY_TRIGGERED` | 已触发 `readonly`，数据库已被切换或保持为 `readonly` |
| `READONLY_FAILED` | 尝试切换 `readonly` 失败 |
| `READWRITE_PENDING` | 磁盘已恢复，但仍处于冷却时间内，暂不自动恢复 `readwrite` |
| `READWRITE_FAILED` | 尝试恢复 `readwrite` 失败 |

## 9. 磁盘对象状态

`STATUS` 字段用于展示每个磁盘对象的采集和告警状态。

| 状态 | 含义 | 是否参与 `readonly` 触发 |
| --- | --- | --- |
| `OK` | 采集成功，磁盘使用率未达到阈值 | 否 |
| `WARN` | 采集成功，磁盘使用率达到或超过阈值 | 是 |
| `ERROR` | 采集失败 | 否 |

说明：

- `WARN` 表示该磁盘对象已经达到保护阈值。
- `ERROR` 表示采集失败，需要检查本地数据库进程、DSS 环境、`DSS_HOME`、`dsscmd` 或权限配置。
- `ERROR` 对象不会作为成功告警对象触发 `readonly`，但需要运维人员排查采集失败原因。

## 10. 自动 readonly 触发逻辑

自动 `readonly` 需要同时满足以下条件：

| 条件 | 说明 |
| --- | --- |
| 自动保护开关开启 | `_DISK_USAGE_PROTECT_ENABLE = TRUE` |
| 巡检线程正常运行 | CMS 磁盘使用率巡检线程已启动 |
| 至少一个对象采集成功 | 本地数据库目录或 DSS VG 至少一个对象采集成功 |
| 磁盘使用率达到阈值 | 任一成功采集对象满足 `USE% >= _DISK_USAGE_THRESHOLD` |
| 当前未处于已触发状态 | 当前没有处于 `READONLY_TRIGGERED` 状态 |
| 满足冷却时间要求 | 距离上一次 `readonly`/`readwrite` 动作已超过 `_DISK_USAGE_READONLY_COOLDOWN` |

满足以上条件后，CMS 会向本节点数据库资源发送 `readmode switch` 消息，将数据库切换为 `readonly`。

## 11. 自动 readwrite 恢复逻辑

自动恢复 `readwrite` 需要同时满足以下条件：

| 条件 | 说明 |
| --- | --- |
| 曾由该功能触发 `readonly` | 数据库 `readonly` 状态由磁盘保护功能触发 |
| 自动保护开关开启 | `_DISK_USAGE_PROTECT_ENABLE = TRUE` |
| 磁盘使用率已恢复 | 成功采集的磁盘对象均未达到阈值 |
| 触发对象已恢复 | 当初触发 `readonly` 的磁盘对象均恢复到阈值以下 |
| 满足冷却时间要求 | 距离上一次 `readonly`/`readwrite` 动作已超过 `_DISK_USAGE_READONLY_COOLDOWN` |
| 数据库状态允许恢复 | 数据库当前状态满足 `readwrite` 恢复条件 |

满足以上条件后，CMS 会自动尝试恢复数据库为 `readwrite`。

如果不希望等待自动恢复，可以在确认磁盘空间已清理且数据库具备恢复条件后执行：

```bash
cms diskreadonly -recover-now
```

## 12. 冷却时间说明

冷却时间用于限制 `readonly`/`readwrite` 动作的执行频率，避免数据库读写状态频繁切换。

### 示例场景

| 时间点 | 操作或状态 | 是否刷新冷却时间 |
| --- | --- | --- |
| `T1` | 磁盘使用率达到阈值，CMS 自动切换数据库为 `readonly` | 是 |
| `T2` | 未超过 `cooldown`，用户手动恢复 `readwrite` | 是 |
| `T3` | 再次检测到磁盘使用率达到阈值 | 需要重新判断是否超过 `T2` 后的 `cooldown` |
| `T4` | 超过 `cooldown` 后仍满足 `readonly` 条件 | 可再次触发 `readonly` |

说明：

- 冷却时间按照最近一次 `readonly`/`readwrite` 动作时间计算。
- 手动恢复 `readwrite` 会刷新最近一次动作时间。
- 未超过 `cooldown` 时，即使磁盘使用率达到阈值，也会进入 `pending` 状态，不会立即切换。
- 该机制用于避免磁盘使用率在阈值附近波动时导致数据库反复切换读写状态。

## 13. 配置文件示例

`cms.ini` 中的相关配置示例：

```ini
_DISK_USAGE_CHECK_INTERVAL = 30
_DISK_USAGE_THRESHOLD = 85
_DISK_USAGE_PROTECT_ENABLE = TRUE
_DISK_USAGE_READONLY_COOLDOWN = 120
```

| 配置项 | 示例值 | 含义 |
| --- | ---: | --- |
| `_DISK_USAGE_CHECK_INTERVAL` | `30` | CMS 每 30 秒检查一次磁盘使用率 |
| `_DISK_USAGE_THRESHOLD` | `85` | 磁盘使用率达到 85% 时触发保护判断 |
| `_DISK_USAGE_PROTECT_ENABLE` | `TRUE` | 开启自动 `readonly`/`readwrite` 保护 |
| `_DISK_USAGE_READONLY_COOLDOWN` | `120` | `readonly`/`readwrite` 动作之间至少间隔 120 秒 |



