# GUC参数

## **_INDEX_BUFFER_SIZE**

**参数描述**  
用于存储索引热点页（当前主要指B树索引根节点页面）只读副本的全局缓存。该缓存旨在减少对高并发访问索引页的争用，从而提升系统并发性能。

**取值范围**

- 最小值：`16K` （16,384 字节）
  
- 最大值：`32T` （35,184,372,088,832 字节）
  
- 单位：字节
  
**默认值**  
`8M` （8,388,608 字节）

## **_OPT_CBO_STAT_SAMPLING_LEVEL**

**参数描述**
设置动态采样等级，用于优化器收集统计信息。该参数决定了优化器在缺失统计信息或统计信息过时的情况下生成执行计划时，是否采用动态采样以及采样的比率大小。

**取值范围**

- 整数

- 最小值：`0`，表示关闭动态采样功能。

- 最大值：`9`

**默认值**
`0`

## **_PREFETCH_ROWS**

**参数描述**  
设置在数据检索过程中，系统提前预读取到缓存中的行数。

**取值范围**

- 整数
  
- 最小值：`1`
  
- 最大值：`2^32 - 1`
  
**默认值**  
`100`

## **CHECKPOINT_PERIOD**

**参数描述**  
设置系统执行两次checkpoint之间的最大时间间隔。当达到此预设的时间间隔时，将自动触发一次checkpoint。

**特性**

- **别名**：`CHECKPOINT_TIMEOUT`（不建议修改）。

**取值范围**

- 整数，单位：秒
  
- 最小值：`1` 
  
- 最大值：`2^32 - 1` 
  
**默认值**  
`300` 

**生效方式**

立即生效（7.0.0-LTS 实测 `EFFECTIVE=immediately`）。7.0.0-LTS 实测 `ALTER SYSTEM SET CHECKPOINT_PERIOD = 0 SCOPE=MEMORY;` 报错：

```
OG-00209, The value of parameter "CHECKPOINT_PERIOD" is too small, at least 1
```

## **DTC_CKPT_NOTIFY_TASK_RATIO**

**参数描述**

设置 DTC（分布式事务集群）检查点通知任务的比例：在 MES 任务线程中分配给检查点通知消息组的线程数量为 `OGRAC_TASK_NUM × 该比例`（向下取整）。比例越高，检查点通知的调度越及时。

**类型与取值范围**：`OG_TYPE_REAL`；取值范围 `[0.001,0.999]`（闭区间）。

**默认值**

- 内核缺省值：0.125。
- 部署默认值：0.032（随包部署工具写入，见 ograc_ctl.py:610）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`。7.0.0-LTS 实测：`ALTER SYSTEM SET` 后 `VALUE` 立即变、`RUNTIME_VALUE` 不变；`deploy stop` 后 `deploy start` 重启，重启后 `RUNTIME_VALUE` 才等于新 `VALUE`。

**配置建议**：请按视图取值区间 `[0.001,0.999]` 设置。7.0.0-LTS 实测越界均被拒，且**报错里显示的名字是 `MES_TASK_RATIO`**（说明二者在引擎内部是同一校验对象，文档宜提示该别名关系）：

```
alter system set DTC_CKPT_NOTIFY_TASK_RATIO = 5;       ->  OG-00202, The parameter value "MES_TASK_RATIO" was invalid
alter system set DTC_CKPT_NOTIFY_TASK_RATIO = 0.0001;  ->  OG-00202, The parameter value "MES_TASK_RATIO" was invalid
```

修改需重启，请安排在维护窗口。上述试设/恢复示例使用 `SCOPE=MEMORY`，仅用于临时验证，不会写入参数配置文件；生产持久调整请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`，并安排重启后在维护窗口确认 `RUNTIME_VALUE` 更新。

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 0.05）
ALTER SYSTEM SET DTC_CKPT_NOTIFY_TASK_RATIO = 0.05 SCOPE=MEMORY;

-- 读回：VALUE 立即为新值，RUNTIME_VALUE 在重启前仍为旧值
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='DTC_CKPT_NOTIFY_TASK_RATIO';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 0.032，请与实际实例核对后再执行）
ALTER SYSTEM SET DTC_CKPT_NOTIFY_TASK_RATIO = 0.032 SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

## **OGRAC_TASK_NUM**

**参数说明**：设置 oGRAC 的 MES 任务线程总数。该值是 MES 任务线程分配比例的基数：检查点通知消息组分配到的线程数为 `OGRAC_TASK_NUM × DTC_CKPT_NOTIFY_TASK_RATIO`（向下取整，见 `DTC_CKPT_NOTIFY_TASK_RATIO` 小节）。

**取值范围**：整数，`[1,500]`（SQL 设置阶段的校验范围；`CLUSTER_DATABASE=TRUE` 时实例启动检查要求 `≥16`，见配置建议）。

**默认值**

- 内核缺省值：`16`。
- 部署默认值：`256`（部署工具写入的常见默认值；小规格部署另有 `64` 档），以实际部署配置为准。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。实测：设为 300 → 重启 → 读回 `RUNTIME_VALUE=300`；还原 256 → 重启 → 读回 256/256。

**配置建议**：本参数是 DTC 系列比例参数的计算基数——调整后 `DTC_CKPT_NOTIFY_TASK_RATIO`、`DTC_CLEAN_EDP_TASK_RATIO`、`DTC_TXN_INFO_TASK_RATIO` 对应的实际线程数会按新基数重新计算，请一并评估；修改需重启，建议在维护窗口进行。

> ⚠️ **SQL 校验范围与启动检查是两道独立约束**：`[1,500]` 是 SQL 设置阶段的校验范围；部署为 `CLUSTER_DATABASE=TRUE` 时，**实例启动检查要求 `OGRAC_TASK_NUM ≥ 16`**。低于 16 的值（如 8）在 SQL 阶段不会被拦截，且默认 scope 会写入参数配置文件——重启后启动检查报该参数无效、实例无法启动。调整下限时不得低于 16，并保留参数配置文件备份以便回滚。

**示例**：

```sql
ALTER SYSTEM SET OGRAC_TASK_NUM = 300;   -- 持久化（默认 SCOPE=BOTH）
-- 重启后读回 300 / 300
ALTER SYSTEM SET OGRAC_TASK_NUM = 256;   -- 还原（部署默认值 256；小规格部署为 64 或按实际原值）
-- 重启后读回 256 / 256
```

## **PAGE_CLEAN_MODE**

**参数描述**

设置页面清理模式。`SINGLE`：逐个 buffer set 清理脏页；`ALL`：一次性清理全部 buffer set 的脏页。`ALL` 清理更彻底但单次停顿更大，`SINGLE` 更平滑。

**类型与取值范围**：`OG_TYPE_VARCHAR`；RANGE 为枚举串 `SINGLE,ALL`。

**默认值**

- 内核缺省值：SINGLE。
- 部署默认值：ALL（随包部署工具写入，见 ograc_ctl.py:682）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`，修改后立即生效，不需要重启。

**配置建议**：取值必须**不带引号**。7.0.0-LTS 实测 `ALTER SYSTEM SET PAGE_CLEAN_MODE = 'SINGLE' SCOPE=MEMORY;`（带单引号）报错：

```
OG-00601, [1:36]Sql syntax error: SINGLE or ALL expected
```

写成 `ALTER SYSTEM SET PAGE_CLEAN_MODE = SINGLE;`（不带引号）则执行成功。上述示例使用 `SCOPE=MEMORY`，修改立即生效但不会持久化，重启后丢失；如需持久化，请省略 scope（默认 `SCOPE=BOTH`）。

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 SINGLE，注意不加引号）
ALTER SYSTEM SET PAGE_CLEAN_MODE = SINGLE SCOPE=MEMORY;

-- 读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='PAGE_CLEAN_MODE';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 ALL，请与实际实例核对后再执行）
ALTER SYSTEM SET PAGE_CLEAN_MODE = ALL SCOPE=MEMORY;
```

## **ENABLE_QUICK_CKPT**

**结论**：该参数在 7.0.0-LTS 安装包中不存在，本节不提供参数定义；待升级到包含提交 `34f2b0f` 的版本后，在对应版本文档中补充定义。

**版本说明**：该参数已在 oGRAC master 源码中引入——`srv_param.c:1228` 注册为 `BOOLEAN` 类型、默认 `FALSE`、重启生效，由 2026-06-21 提交 `34f2b0f feat(gbp): integrate kernel GBP paths` 带入。7.0.0-LTS 安装包版本早于该提交，经实机核查（`SHOW PARAMETER` 无返回、`DV_PARAMETERS` 视图 442 个参数中无该项、引擎二进制 grep 计数为 0），本安装包不含该参数。

## **CHECKPOINT_PAGES**

**参数描述**  
设定触发checkpoint的脏页数量阈值。当数据库缓冲区中的脏页数量达到此设定值时，将自动触发一次checkpoint。

**特性**

- **别名**：`CHECKPOINT_INTERVAL`（不建议修改）。

**取值范围**

- 整数，单位：页
  
- 最小值：`1` 
  
- 最大值：`2^32 - 1` 
  
**默认值**  
`100,000` 

**生效方式**

立即生效（7.0.0-LTS 实测 `EFFECTIVE=immediately`）。7.0.0-LTS 实测 `ALTER SYSTEM SET CHECKPOINT_PAGES = 0 SCOPE=MEMORY;` 报错：

```
OG-00209, The value of parameter "CHECKPOINT_PAGES" is too small, at least 1
```

设置 `4294967296` 报 `OG-00601, Sql syntax error: Convert uint32 failed, the number text is not in the range of uint32, text = 4294967296`，与取值范围上限 `4294967295` 一致。

## **REACTOR_THREADS**

**参数描述**  
设置用于处理I/O侦听的线程数量。这些线程负责接收并分发来自客户端的连接请求和网络数据包。

**配置建议**

- **推荐值**：`OPTIMIZED_WORKER_THREADS / 50`。
  
- **风险提示**：
  
  1. 设置过大会额外消耗CPU、内存及线程资源。在系统资源不足时，可能导致数据库运行异常。
    
  2. 设置值不允许大于Agent线程的总数。
    
**取值范围**  
[1, 10000]

**默认值**  
`1`

## **OPTIMIZED_WORKER_THREADS**

**参数描述**  
设置数据库最优工作线程的数量。此参数决定了系统在会话与线程之间是采用**绑定模式**还是**分离模式**。

**工作模式**

- 当会话数 > OPTIMIZED_WORKER_THREADS，系统切换为会话和线程分离模式，否则为绑定模式。
  
**与 MAX_WORKER_THREADS 的关系**

1. **启动时**：若配置文件中 `OPTIMIZED_WORKER_THREADS > MAX_WORKER_THREADS`，则自动将 `MAX_WORKER_THREADS` 调整为与本参数值相等。
  
2. **启动后**：
  
    - 可在线将其设置为小于或等于当前 `MAX_WORKER_THREADS`的任意值。
      
    - 若尝试设置为大于当前 `MAX_WORKER_THREADS` 的值，系统将报错。
    
3. **重启后**：`MAX_WORKER_THREADS` 将保持原值，不会因本参数的修改而改变。
  
**配置建议与约束**

- **最大值上限**：不建议超过 `SESSIONS` 参数的配置值，否则会造成线程资源闲置浪费。
  
- **资源消耗**：
  
  - 每个工作线程约占用 **0.5 M+** 的内存。
    
  - 设置过大会显著增加 CPU 和内存开销，在资源不足时可能导致数据库异常。
    
- **绝对上限**：因受安全函数内存限制（传入内存上限为2G），本参数最大值为 **10000**。
  
**取值范围**  
[2, 10000]

**默认值**  
`100`

## **MAX_WORKER_THREADS**

**参数描述**  
设置数据库工作线程池的最大容量。

**线程池扩展机制（通用行为说明）**

当活跃会话数超过 `OPTIMIZED_WORKER_THREADS` 时，工作线程数可以从最优值继续扩展，直到达到本参数设定的**最大值**。即可额外扩展的线程数为 `MAX_WORKER_THREADS - OPTIMIZED_WORKER_THREADS`。

**配置建议与约束**

- **最大值参考**：`OPTIMIZED_WORKER_THREADS` 不建议超过 `SESSIONS` 参数的配置值，否则会造成线程资源浪费。
- **资源消耗**：每个工作线程约占用 **0.5 M+** 的内存；设置过大会显著增加 CPU 和线程管理开销，在资源不足时可能导致数据库异常。
- **静态校验范围**：[2, 10000]（cm_defs.h:405-406、set_server.c:145）。

**取值范围**  
[2, 10000]

**默认值**  
`100`

**生效方式**

重启生效（7.0.0-LTS 实测 `EFFECTIVE=reboot`）。通过 `ALTER SYSTEM SET` 修改后，重启数据库实例后 `RUNTIME_VALUE` 更新为新值；上述试设示例使用 `SCOPE=MEMORY`，仅用于临时验证。

**7.0.0-LTS 实测补充**：实例 `OPTIMIZED_WORKER_THREADS = 2000` 时，`ALTER SYSTEM SET MAX_WORKER_THREADS = 1000 SCOPE=MEMORY;` 被拒绝：

```
OG-00209, The value of parameter "MAX_WORKER_THREADS" is too small, at least 2000
```

即该版本存在动态下限（实测不低于当时的 `OPTIMIZED_WORKER_THREADS` 值），该动态下限的来源在 master 源码中未见对应实现（master 仅做静态范围校验），**来源待确认**。另设置 `10001` 报 `OG-00202, The parameter value "MAX_WORKER_THREADS" was invalid`，与取值范围上限 `10000` 一致。

## **AUTONOMOUS_SESSIONS**

**参数描述**

设置自治会话数量。当前版本该参数仅保存与校验：可以设置、可以读回、重启后保留，但内核尚未消费该值，调整不影响运行时行为；后续版本启用后请重新确认行为。

**类型与取值范围**

`OG_TYPE_INTEGER`；取值范围 `(0,256]`（左开右闭，即大于 0 且不超过 256）。

**默认值**

8。

**生效方式**

重启生效（7.0.0-LTS 实测 `EFFECTIVE=reboot`）。通过 `ALTER SYSTEM SET` 修改后，重启数据库实例后 `RUNTIME_VALUE` 更新为新值。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议与示例**

`SHOW PARAMETER` 按名称模糊匹配，查询 `AUTONOMOUS_SESSIONS` 时会同时列出 `KNL_AUTONOMOUS_SESSIONS`；两者是相互独立的参数。设置 `KNL_AUTONOMOUS_SESSIONS` 越界时报错信息中出现的是 `AUTONOMOUS_SESSIONS`，因两者共用同一校验函数。越界校验证据：

```
alter system set AUTONOMOUS_SESSIONS = 9999;  ->  OG-00210, The value of parameter "AUTONOMOUS_SESSIONS" is too large, at most 256
```

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 16）
ALTER SYSTEM SET AUTONOMOUS_SESSIONS = 16 SCOPE=MEMORY;

-- 读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='AUTONOMOUS_SESSIONS';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 8，请与实际实例核对后再执行）
ALTER SYSTEM SET AUTONOMOUS_SESSIONS = 8 SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

## **KNL_AUTONOMOUS_SESSIONS**

**参数描述**

设置内核侧自治会话数量。当前版本该参数仅保存与校验：可以设置、可以读回、重启后保留，但内核尚未消费该值，调整不影响运行时行为；后续版本启用后请重新确认行为。

**类型与取值范围**

`OG_TYPE_INTEGER`；取值范围 `(0,256]`（左开右闭，即大于 0 且不超过 256）。

**默认值**

8。

**生效方式**

重启生效（7.0.0-LTS 实测 `EFFECTIVE=reboot`）。通过 `ALTER SYSTEM SET` 修改后，重启数据库实例后 `RUNTIME_VALUE` 更新为新值。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议与示例**

`SHOW PARAMETER` 按名称模糊匹配，查询 `KNL_AUTONOMOUS_SESSIONS` 时会同时列出 `AUTONOMOUS_SESSIONS`；两者是相互独立的参数。设置 `KNL_AUTONOMOUS_SESSIONS` 越界时报错信息中出现的是 `AUTONOMOUS_SESSIONS`，因两者共用同一校验函数。越界校验证据：

```
alter system set KNL_AUTONOMOUS_SESSIONS = 9999;  ->  OG-00210, The value of parameter "AUTONOMOUS_SESSIONS" is too large, at most 256
```

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 16）
ALTER SYSTEM SET KNL_AUTONOMOUS_SESSIONS = 16 SCOPE=MEMORY;

-- 读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='KNL_AUTONOMOUS_SESSIONS';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 8，请与实际实例核对后再执行）
ALTER SYSTEM SET KNL_AUTONOMOUS_SESSIONS = 8 SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

## **NORMAL_USER_RESERVED_SESSIONS_FACTOR**

**参数描述**

设置普通用户可占用的超级用户预留会话比例：普通用户最多可使用 `SUPER_USER_RESERVED_SESSIONS × 该值`（向下取整）个预留会话。取值 `1` 表示普通用户可占满全部预留会话，`0` 表示普通用户不可使用预留会话。

**与 SUPER_USER_RESERVED_SESSIONS 的关系**：该参数作用于超级用户预留会话（`SUPER_USER_RESERVED_SESSIONS` 的取值与生效方式见其自身条目）；实际可用的预留会话数随两者配置联动计算。

**类型与取值范围**：`OG_TYPE_REAL`；取值范围 `[0,1]`（闭区间）。

**默认值**：1（7.0.0-LTS 实例 `DV_PARAMETERS.DEFAULT_VALUE`）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`，修改后立即生效，不需要重启。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议**：取值范围为 `[0,1]`，超出该区间会被拒绝。7.0.0-LTS 实测 `ALTER SYSTEM SET NORMAL_USER_RESERVED_SESSIONS_FACTOR = 2 SCOPE=MEMORY;` 报错：

```
OG-00202, The parameter value "NORMAL_USER_RESERVED_SESSIONS_FACTOR" was invalid
```

（视图取值范围 `[0,1]` 可解释该报错。）

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 0.5）
ALTER SYSTEM SET NORMAL_USER_RESERVED_SESSIONS_FACTOR = 0.5 SCOPE=MEMORY;

-- 读回：该参数为 immediately 生效，VALUE 与 RUNTIME_VALUE 应同时为新值
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='NORMAL_USER_RESERVED_SESSIONS_FACTOR';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 1，请与实际实例核对后再执行）
ALTER SYSTEM SET NORMAL_USER_RESERVED_SESSIONS_FACTOR = 1 SCOPE=MEMORY;
```

## **OPEN_CURSORS**

**参数描述**：设置单个会话可同时打开的游标数量上限（视图 `RANGE=[1,16384]`）。

**类型与取值范围**：`OG_TYPE_INTEGER`；取值范围 `[1,16384]`。

**默认值**：2000（7.0.0-LTS 实例 `DV_PARAMETERS.DEFAULT_VALUE`）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`，修改后立即生效，不需要重启。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议**：上限为 16384。7.0.0-LTS 实测 `ALTER SYSTEM SET OPEN_CURSORS = 99999999 SCOPE=MEMORY;` 报错：

```
OG-00210, The value of parameter "OPEN_CURSORS" is too large, at most 16384
```

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 3000）
ALTER SYSTEM SET OPEN_CURSORS = 3000 SCOPE=MEMORY;

-- 读回：该参数为 immediately 生效，VALUE 与 RUNTIME_VALUE 应同时为新值
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='OPEN_CURSORS';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 2000，请与实际实例核对后再执行）
ALTER SYSTEM SET OPEN_CURSORS = 2000 SCOPE=MEMORY;
```

## **STATS_LEVEL**

**参数描述**  
控制是否收集并统计对表进行的DML（数据操作语言）操作信息。

**特性**

- **别名**：`STATISTICS_LEVEL`（不建议修改）。

**取值范围与行为**

| 取值  | 含义与行为 |
|  |  |
| **TYPICAL** 或 **ALL** | 启用表监控（Table-monitoring）。<br>• 系统将收集对表执行的DML操作的统计信息。<br>• 收集到的统计信息会在约 15分钟后展示在系统表 `SYS_DML_STATS`中。 |
| **BASIC** | 禁用Table-monitoring。<br>• 不收集上述DML操作统计信息。 |

**默认值**  
`TYPICAL`

## use\_bison\_parser<a name="use_bison_parser"></a>

**参数说明**：是否使用Bison解析器解析普通SQL语句。

该参数设置后立即生效，请参考[配置数据库系统参数](../database_administration_guide/basic_management_of_database_system/configuring_the_database_system.md#配置数据库系统参数)中对应设置方法进行设置。

**取值范围**：布尔型

- `true`表示使用Bison解析器。
- `false`表示使用原生解析器。
- `on`/`off`仅在Bison解析器已启用时可用于设置该参数。

**默认值**：false

>[!NOTE]说明
>
>普通SQL语句中，以f/d结尾的浮点数字面量由该参数控制。

## **AUTO_INHERIT_USER**

**参数描述**

控制创建用户时是否自动向 `PUBLIC` 授予对新用户的 `INHERIT PRIVILEGES` 权限。`ON` 会执行该授权；`OFF` 不执行。此参数影响后续创建的用户，不应将切换参数视为撤销已有授权。

**类型与取值范围**

`OG_TYPE_VARCHAR`；`OFF`、`ON`。

**默认值**

`OFF`。

**生效方式**

立即生效。

**配置建议**

一般保持 `OFF`。`PUBLIC` 代表所有数据库用户，开启自动授权前应审查权限继承需求及现有授权策略。可通过 `DV_PARAMETERS` 查询参数状态；仅设置成功不能证明实际权限授予行为。

## **BACKUP_RETRY**

**参数描述**

控制数据库备份在满足内核重试条件时是否尝试重试。源码中的重试路径还会检查失败原因、清理待重试备份集并等待集群 reform 完成；该参数不表示所有备份失败都会自动重试。

**类型与取值范围**

`OG_TYPE_BOOLEAN`；`FALSE`、`TRUE`。

**默认值**

`TRUE`。

**生效方式**

`DV_PARAMETERS.EFFECTIVE` 标记为 `immediately`。设置及读回成功不能证明实际备份重试路径已采用新值，还需在具备重试条件的隔离环境做功能验证。

**配置建议与示例**

通常保持 `TRUE`。以下示例仅用于无备份任务时验证设置接口，假设原值为 `TRUE`；验证后立即恢复，不代表已完成备份故障重试测试。

```sql
ALTER SYSTEM SET BACKUP_RETRY = FALSE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BACKUP_RETRY';
ALTER SYSTEM SET BACKUP_RETRY = TRUE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BACKUP_RETRY';
```

## **BATCH_FLUSH_CAPACITY**

**参数描述**

仅在启用 DBStor（`ENABLE_DBSTOR=TRUE`）且启用 DBStor 批量刷盘（`ENABLE_DBSTOR_BATCH_FLUSH=TRUE`）时，限制数据库写入线程在该刷盘路径中一批异步脏页写入的数量。文件系统等未进入此路径的部署，不应把它当作通用的检查点刷盘调节参数。增大该值可能改变该路径的单批 I/O 请求规模，实际效果取决于工作负载和存储能力。

**类型与取值范围**

`OG_TYPE_INTEGER`；整数 `[1, 4096]`，单位为页。

**默认值**

`160`。

**生效方式**

重启生效。设置接口返回成功后，应分别查看 `VALUE` 与 `RUNTIME_VALUE`；配置值变化不能证明当前写入线程已采用新批次大小。

**配置建议与示例**

未启用 DBStor 批量刷盘时保持默认值；在启用该功能的部署中，调整前应评估存储 I/O 和内存负载。生产持久调整需通过 `SCOPE=PFILE` 写入配置并安排重启窗口。以下是在 7.0.0 单节点文件系统实例上验证过的临时设置接口示例，假设原值为 `160`；示例会恢复原值，不重启数据库，也未验证 DBStor 批量刷盘效果。

```sql
ALTER SYSTEM SET BATCH_FLUSH_CAPACITY = 161 SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BATCH_FLUSH_CAPACITY';
ALTER SYSTEM SET BATCH_FLUSH_CAPACITY = 160 SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BATCH_FLUSH_CAPACITY';
```

## **BLOCK_REPAIR_ENABLE**

**参数描述**

控制自动块修复开关。源码中的修复任务还要求存在有效备节点，并受 `BLOCK_REPAIR_TIMEOUT` 限制；单独打开开关不保证坏块可以修复。

**类型与取值范围**

`OG_TYPE_BOOLEAN`；`FALSE`、`TRUE`。

**默认值**

`FALSE`。

**生效方式**

立即生效。

**配置建议与示例**

仅在具备可用备节点、已确认修复流程和超时设置时考虑启用。单节点环境无法验证自动块修复功能。以下示例假设原值为 `FALSE`，只验证参数设置和读回，不制造坏块。

```sql
ALTER SYSTEM SET BLOCK_REPAIR_ENABLE = TRUE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BLOCK_REPAIR_ENABLE';
ALTER SYSTEM SET BLOCK_REPAIR_ENABLE = FALSE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BLOCK_REPAIR_ENABLE';
```

## **BUF_POOL_NUM**

**参数描述**

设置数据缓冲区的缓冲池分区数量。启动时若 `DATA_BUFFER_SIZE` 不足以让每个配置分区达到 256 MiB，内核会降低实际分区数，因此参数配置值不一定等于最终创建的分区数。

**类型与取值范围**

`OG_TYPE_INTEGER`；整数 `[1, 4096]`。`0` 不属于 oGRAC 7.0.0 的合法值。

**默认值**

oGRAC 7.0.0 内核注册默认值为 `1`。安装器或管理员可能显式写入同一个数值；即使 `VALUE`、`RUNTIME_VALUE` 和 `DEFAULT_VALUE` 都为 `1`，`ISDEFAULT` 也可能为 `FALSE`，应另查配置文件。

**生效方式**

重启生效。仅设置成功或 `VALUE` 改变，不能证明当前缓冲池数量已改变。

**配置建议与示例**

与 `DATA_BUFFER_SIZE` 和服务器可用内存一起规划。资源有限时不要照搬大型性能测试中的高分区数。生产持久调整需按 `SCOPE=PFILE` 和数据库重启流程处理。修改前先查询当前实例的 `VALUE`、`RUNTIME_VALUE`，并核对持久化配置中的原值；下方恢复为 `1` 仅适用于原配置值为 `1` 的实例。若原值不是 `1`，须将恢复命令中的 `1` 换成事先记录的原值。以下是在 7.0.0 实例上验证过的临时设置接口示例，不重启或重新分配缓冲池。

```sql
SELECT NAME, VALUE, RUNTIME_VALUE, DEFAULT_VALUE FROM DV_PARAMETERS WHERE NAME = 'BUF_POOL_NUM';
ALTER SYSTEM SET BUF_POOL_NUM = 2 SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BUF_POOL_NUM';
-- 仅当修改前配置值为 1 时执行；否则改用记录的原值。
ALTER SYSTEM SET BUF_POOL_NUM = 1 SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BUF_POOL_NUM';
```

## **BUILD_DATAFILE_PREALLOCATE**

**参数描述**

控制创建、扩展或恢复数据文件时是否向底层文件操作传递预分配标志。仅切换参数不会自行创建或扩展数据文件。

**类型与取值范围**

`OG_TYPE_BOOLEAN`；`FALSE`、`TRUE`。

**默认值**

`FALSE`。

**生效方式**

立即生效。

**配置建议与示例**

启用前应核对目标文件系统的可用空间及数据文件增长计划；预分配方式的实际磁盘效果还需结合文件系统和具体文件操作验证。以下示例假设原值为 `FALSE`，不执行任何数据文件操作。

```sql
ALTER SYSTEM SET BUILD_DATAFILE_PREALLOCATE = TRUE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BUILD_DATAFILE_PREALLOCATE';
ALTER SYSTEM SET BUILD_DATAFILE_PREALLOCATE = FALSE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'BUILD_DATAFILE_PREALLOCATE';
```

## **COMMIT_MODE**

**参数描述**

控制提交路径中重做日志刷新请求的调度方式。在会话采用 `COMMIT_WAIT_LOGGING=WAIT` 且需要刷新日志时，`IMMEDIATE` 不额外等待合并刷新，`BATCH` 可短暂等待以合并请求；若会话采用 `NOWAIT`，日志提交函数在这段调度逻辑之前返回，`IMMEDIATE` 也不会因此立即发起刷盘。

**类型与取值范围**

`OG_TYPE_VARCHAR`；`BATCH`、`IMMEDIATE`。

**默认值**

`IMMEDIATE`。

**生效方式**

`DV_PARAMETERS.EFFECTIVE` 标记为 `immediately`。执行 `ALTER SYSTEM SET` 时，通知函数更新全局值和执行设置的当前会话；之后真正新初始化的会话继承全局值，已存在的其他会话保留各自原有设置。空闲会话复用时不会重新读取该值，因此仅重新连接也不能保证采用新值；应按实际会话核对，不能只凭 `RUNTIME_VALUE` 判断所有会话已切换。

**配置建议与示例**

通常保持 `IMMEDIATE`，并保持 `COMMIT_WAIT_LOGGING=WAIT`。调整前应评估事务延迟、吞吐与故障时的数据持久性要求。以下示例假设原值为 `IMMEDIATE`，只在没有业务事务的受控窗口验证设置接口，随后立即恢复；它不能证明提交性能或持久性。

```sql
ALTER SYSTEM SET COMMIT_MODE = BATCH SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'COMMIT_MODE';
ALTER SYSTEM SET COMMIT_MODE = IMMEDIATE SCOPE=MEMORY;
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME = 'COMMIT_MODE';
```

## **COMMIT_ON_DISCONNECT**

**参数描述**

控制会话失败处理路径中的事务处置：`TRUE` 时该路径尝试调用提交；`FALSE` 时，若会话仍有未结束事务且并非强制终止，则调用回滚。具体结果还取决于此前的处理步骤：例如等待客户端连接失败时，源码先回滚，再进入会话失败处理，因此即使设为 `TRUE`，也不能提交已回滚的事务。该参数不保证断连时一律提交，不能替代应用显式执行 `COMMIT` 或 `ROLLBACK`。

**类型与取值范围**

`OG_TYPE_BOOLEAN`；`FALSE`、`TRUE`。

**默认值**

`FALSE`。

**生效方式**

`DV_PARAMETERS.EFFECTIVE` 标记为 `reboot`。源码同时存在更新运行开关的通知函数；因此不能仅凭设置成功或元数据推断断连事务行为，变更时应在隔离环境核对运行值并做事务测试。

**配置建议**

保持 `FALSE`，由应用显式决定事务提交或回滚。开启 `TRUE` 可能提交应用原本希望回滚的未完成事务，应先评估一致性风险。生产环境变更需安排维护及恢复方案。

## **COMMIT_WAIT_LOGGING**

**参数描述**

控制提交时是否等待重做日志刷新完成。`WAIT` 等待日志写入确认；`NOWAIT` 可在等待前返回，因此实例异常时可能丢失尚未持久化的已提交事务。

**类型与取值范围**

`OG_TYPE_VARCHAR`；`WAIT`、`NOWAIT`。

**默认值**

`WAIT`。

**生效方式**

`DV_PARAMETERS.EFFECTIVE` 标记为 `immediately`。执行 `ALTER SYSTEM SET` 时，通知函数更新全局值和执行设置的当前会话；之后真正新初始化的会话继承全局值，已存在的其他会话保留各自原有设置。空闲会话复用时不会重新读取该值，因此仅重新连接也不能保证采用新值；应按实际会话核对，不能只凭 `RUNTIME_VALUE` 判断所有会话已切换。

**配置建议**

一般保持 `WAIT`。只有明确接受相应持久性风险并完成隔离环境验证时才考虑 `NOWAIT`；调整 `COMMIT_MODE` 时也应同时核对本参数。未执行提交事务的参数设置测试不能证明日志持久性。

## **CONTROL_FILES**

**参数描述**

指定数据库控制文件的路径列表。启动加载控制文件时，内核从该配置读取文件名，并尝试打开、读取和校验已有控制文件；若没有配置或所有文件均不可用，数据库无法按正常路径加载控制文件。

**类型与取值范围**

`OG_TYPE_VARCHAR`。文件系统部署可采用括号包围、逗号分隔的路径列表，例如 `(/data/ograc/ctrl1, /data/ograc/ctrl2, /data/ograc/ctrl3)`。路径应指向该实例实际使用的控制文件，并保证运行用户有相应访问权限。`DV_PARAMETERS.RANGE` 显示为 `-`，不代表可使用任意无效路径。

**默认值**

内核注册的默认字符串为空；这不是可直接启动数据库的控制文件配置。实际路径由安装或初始化配置提供，应查询当前实例的配置文件和 `DV_PARAMETERS` 确认。

**生效方式**

`DV_PARAMETERS.MODIFIABLE=FALSE`，不支持通过 `ALTER SYSTEM SET` 修改；视图标记为 `reboot`。不得为了验证设置接口而修改正在使用的路径。

**配置建议**

保持现有有效配置。涉及控制文件迁移或恢复时，应先制定完整的备份、路径校验、停机和回退方案，并在维护窗口按专门流程执行；仅查询或截图不需要改动文件。

## `BUFFER_PAGE_CLEAN_PERIOD`

**参数说明**：控制 Page Clean 定时任务的执行间隔（单位：秒）。  
Page Clean 是一种快速清理 Buffer 中脏页的机制。在常规业务场景中，通常无需启用该定时任务，脏页清理可由 Buffer 淘汰机制自动触发。但在如 TPC-C、Sysbench 等持续高负载的性能测试场景中，若磁盘 I/O 能力较强，可开启此定时任务以加速脏页刷盘。

**取值范围**：整数，[0, 2³² − 1]  
**默认值**：0（表示不启用定时清理）

## `INTERCONNECT_CHANNEL_NUM`

**参数说明**：配置内部通信（MES）连接通道的数量。  
**取值范围**：整数，[1, 32]  
**默认值**：4

## `INTERCONNECT_TYPE`

**参数说明**：设置节点间通信（MES）使用的协议类型。取值 `TCP`、`UC`、`UC_RDMA`，分别对应 TCP 传输、统一通信协议、统一通信协议 + RDMA；其中 `UC` 在节点内进程间使用共享内存、节点间使用 TCP。

**取值范围**：`TCP`、`UC`、`UC_RDMA`（参数视图 `RANGE` 为 `TCP,UC,UC_RDMA`）。

**默认值**

- 内核缺省值：`TCP`。
- 安装部署值随部署形态而定：随附的《数据库参数配置说明》记载"默认值 TCP，安装默认值为 UC"（见 `database_administration_guide/data_object_management/parameter_description.md` 的 INTERCONNECT_TYPE 条目），请以实际配置文件为准。

**生效方式**：重启生效（参数视图 `EFFECTIVE=reboot`）。该参数**不能通过 SQL 修改**：执行 `ALTER SYSTEM SET` 会被拒绝（报错原文见下）。修改须通过集群配置文件完成并重启。

```sql
ALTER SYSTEM SET INTERCONNECT_TYPE = 'UC';
OG-00219, The parameter "INTERCONNECT_TYPE" is not modifiable with this option
```

> 说明：参数视图中该参数的 `MODIFIABLE` 为 `TRUE`，但 SQL 修改实际被拒——`MODIFIABLE` 列不能单独作为"可通过 SQL 修改"的判据，请以实测与配置方式为准。

**配置建议**：多节点部署时各节点的协议类型须保持一致；协议变更影响节点间传输行为，修改前确认所有节点支持目标协议，并安排在维护窗口重启。

## `LSNR_ADDR`

**参数说明**：设置数据库监听地址，支持配置多个地址（英文逗号分隔，最多 8 个）。除配置文件外，还支持通过 `ALTER SYSTEM ADD/DELETE LSNR_ADDR` 语法动态增删监听地址（见 `sql_reference/alter_system.md` 的 LSNR_ADDR 条目）。

**取值范围**：有效的 IP 地址或域名，多地址用英文逗号分隔；参数视图未提供范围。

**默认值**：`127.0.0.1`。

**生效方式**：参数视图为 `EFFECTIVE=reboot`（配置文件修改后重启生效）。另有一条**动态语法路径**：`ALTER SYSTEM ADD/DELETE LSNR_ADDR` 执行后立即生效——参数读回值即时更新，`ss -ltnp` 可观察到新地址的监听套接字，无需重启。两条路径的语义不同：前者改的是参数值（重启后生效）；后者是对监听地址列表的增删，默认同时写入参数配置文件，增删后立即生效。

**配置建议**：

- 增加一个不存在于本机网卡的 IP 会直接报错（`OG-00259`）。
- 删除正在使用的监听地址会导致相关连接中断。
- `ALTER SYSTEM ADD/DELETE LSNR_ADDR` 默认按持久化处理：既更新运行时监听，也同步写入参数配置文件，一般无需再手工修改配置文件；需要回滚时反向执行 `DELETE` / `ADD` 即可。

**示例**（`192.0.2.0/24` 为文档示例网段）：

```sql
-- 边界：非本机网卡地址被拒绝
ALTER SYSTEM ADD LSNR_ADDR '192.0.2.77';
-- OG-00259, IP address 192.0.2.77 is not local ip, please check your ifconfig

-- 动态添加本机网卡地址（立即生效）
ALTER SYSTEM ADD LSNR_ADDR '<本机网卡IP>';
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='LSNR_ADDR';
-- VALUE = RUNTIME_VALUE = 127.0.0.1,<本机网卡IP>
-- ss -ltnp 可同时看到 127.0.0.1:1611 与 <本机网卡IP>:1611 两个监听

-- 动态删除（还原）
ALTER SYSTEM DELETE LSNR_ADDR '<本机网卡IP>';
```

## `LSNR_PORT`

**参数说明**：设置数据库监听端口。

**取值范围**：整数，`[1024,65535]`。

**默认值**：`1611`。

**生效方式**：重启生效（`EFFECTIVE=reboot`）。两种 scope 的语义不同：

- `SCOPE=MEMORY`：只改内存中的参数值（`VALUE=1612`、`RUNTIME_VALUE=1611`），**不写入参数配置文件，重启后不应用**（仍使用原来的 1611）。
- 默认 scope（`SCOPE=BOTH`）或 `SCOPE=PFILE`：写入参数配置文件，**重启后 `RUNTIME_VALUE` 才等于新端口**，届时需同步调整客户端连接端口。

生产环境修改请在维护窗口进行，修改前备份参数配置文件。

**配置建议**：

- 修改监听端口会导致既有连接全部需要改用新端口，请安排在维护窗口。
- 修改前确认新端口未被其他进程占用，且与集群内其他通信端口不冲突。

**示例**：

```sql
-- 语义演示（SCOPE=MEMORY：只改内存值，配置文件不变，重启后仍为 1611）
ALTER SYSTEM SET LSNR_PORT = 1612 SCOPE=MEMORY;
SELECT NAME, DATATYPE, RANGE, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='LSNR_PORT';
-- VALUE = 1612, RUNTIME_VALUE = 1611

-- 持久化修改（默认 SCOPE=BOTH：写入配置文件，重启后生效）
ALTER SYSTEM SET LSNR_PORT = 1612;
-- 重启（deploy stop / deploy start）后 RUNTIME_VALUE = 1612，同时调整客户端连接端口

-- 还原（默认 SCOPE=BOTH，重启后恢复 1611）
ALTER SYSTEM SET LSNR_PORT = 1611;
```

## `OG_CLUSTER_STRICT_CHECK`

**参数说明**：设置集群状态的严格检查开关。开启后对集群状态执行严格校验。

**取值范围**：布尔值，`FALSE` / `TRUE`。

**默认值**

- 内核缺省值：`TRUE`。
- 部署默认值：`TRUE`（安装/预装路径默认值；实际以部署配置为准）。

**生效方式**：立即生效（`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`）。实测：设为 TRUE → 立即读回 `VALUE=RUNTIME_VALUE=TRUE`；还原 FALSE → 立即读回 FALSE/FALSE，无需重启。

**配置建议**：关闭严格检查会放宽集群状态校验；单节点部署与多节点部署的校验行为可能不同，调整前请确认部署形态与业务要求。修改立即生效，请评估对在线业务的影响。本次验证实例的当前值为 `FALSE`（取值来源未核实，仅作实例状态参考）。

**示例**：

```sql
ALTER SYSTEM SET OG_CLUSTER_STRICT_CHECK = TRUE;    -- 立即生效
SELECT NAME, DATATYPE, RANGE, DEFAULT_VALUE, VALUE, RUNTIME_VALUE
  FROM DV_PARAMETERS WHERE NAME='OG_CLUSTER_STRICT_CHECK';
-- DEFAULT_VALUE = TRUE, VALUE = RUNTIME_VALUE = TRUE

ALTER SYSTEM SET OG_CLUSTER_STRICT_CHECK = FALSE;   -- 还原为本次验证前的取值（该实例为 FALSE）
```

## `ALARM_LOG_DIR`

**参数说明**：设置告警日志的存储目录。告警日志文件名为 `<实例名>_alarm.log`（取值来自 `INSTANCE_NAME` 参数：内核缺省值为 `oGRAC`，部署脚本默认写入 `ograc`）；参数未配置时，告警日志写入数据目录下的 `log` 子目录。告警日志的开启条件（`_LOG_LEVEL` 大于 0）与命名规则见《日志管理》的 ALARM_LOG_DIR 条目（`database_administration_guide/data_object_management/log_management.md`）。

**取值范围**：目录路径字符串；参数视图未提供范围。

**默认值**：（空）——为空时使用上述默认目录。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。实测：设置为已存在的目录 → 重启 → 读回 `VALUE=RUNTIME_VALUE=<该目录>`，配置已应用。

**配置建议**：

- 路径**必须已存在**：指向不存在的目录会被拒绝（`OG-00601 ... is not an existing folder`）。
- 该参数**不接受空值**：以空值设置会被拒绝；如需恢复默认路径，请删除配置文件中的该键并重启。
- 目录须保证 oGRAC 运行用户有写权限；变更后建议重启后确认告警文件在新目录生成。

**示例**：

```sql
ALTER SYSTEM SET ALARM_LOG_DIR = '/path/to/alarm_log_dir';   -- 持久化（默认 SCOPE=BOTH）
-- 重启后读回 VALUE = RUNTIME_VALUE = /path/to/alarm_log_dir
```

**边界证据**：

```sql
ALTER SYSTEM SET ALARM_LOG_DIR = '';
OG-00601, [1:34]Sql syntax error: " is not an existing folder
```

## `LOG_BUFFER_COUNT`

**参数说明**：设置 Log Buffer 的数量。  
**取值范围**：整数，(0, 16]  
**默认值**：4  

**生效方式**

重启生效（7.0.0-LTS 实测 `EFFECTIVE=reboot`）。执行 `ALTER SYSTEM SET` 后 `VALUE` 立即改变、`RUNTIME_VALUE` 不变，重启数据库后 `RUNTIME_VALUE` 才等于新的 `VALUE`。

## `LOG_BUFFER_SIZE`

**参数说明**：定义每个 Log Buffer 的大小，用于缓存 REDO 日志。  
适当增大该值有助于提升数据写入性能，应根据实际可用内存进行调整。

**取值范围**：整数，[1M, 110M]（单位：字节；以 7.0.0-LTS 实测为准）  
master 源码上限已放宽至 1024M（cm_defs.h:568），升级到包含该改动的版本后请以实际版本为准。  
**默认值**：4M  

**生效方式**

重启生效（7.0.0-LTS 实测 `EFFECTIVE=reboot`）。7.0.0-LTS 实测设置 `128M` 被拒：

```
OG-00601, [1:36]Sql syntax error: size value is bigger than maximum(115343360) required
```

即 7.0.0-LTS 实测上限为 `115343360` 字节（`110M`）。

## `TIMED_STATS`（别名：`TIMED_STATISTICS`，不推荐使用）

**参数说明**：控制是否收集与时间相关的统计信息。  

- **TRUE**：收集并存储时间相关的统计信息（可查询 `DV_SYS_STATS` 动态视图或跟踪文件）  
- **FALSE**：所有时间相关统计值置为 0  

**默认值**：TRUE

## `SQL_STAT`

**参数说明**：控制 SQL 性能统计视图的开关，参数说明范围为整个数据库实例。  

- **TRUE**：开启 SQL 性能统计  
- **FALSE**：关闭 SQL 性能统计  

**默认值**：TRUE

## `MES_ELAPSED_SWITCH`

**参数说明**：是否启用 MES 内部通信的时延统计功能。

- **TRUE**：开启时延统计
- **FALSE**：关闭时延统计

**取值范围**：布尔值，`FALSE` / `TRUE`。

**默认值**：`TRUE`。

**生效方式**：立即生效（`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`）。实测：设置为 FALSE 后立即读回 `VALUE=RUNTIME_VALUE=FALSE`；还原 TRUE 后立即读回 TRUE/TRUE，无需重启。

**配置建议**：默认开启（内核缺省值 `TRUE`），持续采集 MES 内部通信时延；无需统计时可关闭以降低开销，关闭后相关时延统计不再采集。调整立即生效，不影响已有连接。

**示例**：

```sql
ALTER SYSTEM SET MES_ELAPSED_SWITCH = FALSE;   -- 立即生效
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='MES_ELAPSED_SWITCH';
-- FALSE / FALSE

ALTER SYSTEM SET MES_ELAPSED_SWITCH = TRUE;    -- 还原
```


## `MES_CHANNEL_DEGRADE_TIME_MS`

**参数说明**：设置 MES 通道降级判定的持续时间（毫秒）。默认值 `0` 表示使用引擎默认的 subhealth 判定阈值；当本参数与 `MES_CHANNEL_UPGRADE_TIME_MS` **两者均非 0** 时，毫秒值才换算为纳秒并作为自定义阈值生效（源码 `set_subhealth_threshold`）——仅设置其中一个、另一个仍为 0 时自定义阈值不生效。UC 传输侧的通道行为细节依赖实现，待确认。

**取值范围**：整数，单位为毫秒；参数视图未提供范围。

**默认值**：`0`。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。实测：设为 5000 → `deploy stop` / `deploy start` 重启 → 读回 `VALUE=RUNTIME_VALUE=5000`；还原为 0 → 重启 → 读回 0/0。注意两者的区别：**参数值本身被加载 ≠ 自定义阈值启用**——按上节语义，若另一个通道参数仍为 0，自定义阈值不生效。默认 scope（`SCOPE=BOTH`）下 `ALTER SYSTEM SET` 既更新内存中的参数值（`VALUE` 立即变化）又写入参数配置文件；本参数为重启生效，`RUNTIME_VALUE` 在重启后才更新为新值。

**配置建议**：参数视图未提供取值区间，调整前请在测试环境验证；单节点部署下 MES 通道的降级行为与多节点部署可能不同，无明确需求时建议保持默认值。

**示例**：

```sql
ALTER SYSTEM SET MES_CHANNEL_DEGRADE_TIME_MS = 5000;   -- 省略 scope：默认 SCOPE=BOTH，写入配置文件
-- 执行 deploy stop / deploy start 重启后读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='MES_CHANNEL_DEGRADE_TIME_MS';
-- 5000 / 5000

ALTER SYSTEM SET MES_CHANNEL_DEGRADE_TIME_MS = 0;      -- 还原
-- 重启后读回 0 / 0
```

## `MES_CHANNEL_UPGRADE_TIME_MS`

**参数说明**：设置 MES 通道升级判定的持续时间（毫秒）。默认值 `0` 表示使用引擎默认的 subhealth 判定阈值；与 `MES_CHANNEL_DEGRADE_TIME_MS` **两者均非 0** 时自定义阈值才生效（源码 `set_subhealth_threshold`），仅设置其中一个不生效。UC 传输侧的通道行为细节依赖实现，待确认。

**取值范围**：整数，单位为毫秒；参数视图未提供范围。

**默认值**：`0`。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。实测：设为 5000 → 重启 → 读回 `VALUE=RUNTIME_VALUE=5000`；还原为 0 → 重启 → 读回 0/0。注意两者的区别：**参数值本身被加载 ≠ 自定义阈值启用**——按上节语义，若另一个通道参数仍为 0，自定义阈值不生效。默认 scope（`SCOPE=BOTH`）下 `ALTER SYSTEM SET` 既更新内存中的参数值（`VALUE` 立即变化）又写入参数配置文件；本参数为重启生效，`RUNTIME_VALUE` 在重启后才更新为新值。

**配置建议**：同 `MES_CHANNEL_DEGRADE_TIME_MS`。

**示例**：

```sql
ALTER SYSTEM SET MES_CHANNEL_UPGRADE_TIME_MS = 5000;   -- 持久化（默认 SCOPE=BOTH）
-- 重启后读回 5000 / 5000
ALTER SYSTEM SET MES_CHANNEL_UPGRADE_TIME_MS = 0;      -- 还原
```

## `MES_CPU_INFO`

**参数说明**：设置 MES 模块使用的 CPU 信息。**可接受的取值格式未确认**：7.0.0-LTS 实测中，形如 `1-3` 的 CPU 区间与任意字符串均被拒绝（报错原文见下），该参数接受的格式需结合产品实现确认；本节仅描述参数配置接口层面的实测结果。

**取值范围**：`OG_TYPE_VARCHAR`；参数视图未提供范围。

**默认值**：（空）。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。

**配置建议**：可接受格式未确认前，请勿在生产环境调整。实例启动日志中同类参数 `CPU_GROUP_INFO` 的取值为 `1-3` 形式，可供参考，但同一形式用于本参数仍被拒绝。

**边界证据**：

```sql
ALTER SYSTEM SET MES_CPU_INFO = '1-3' SCOPE=MEMORY;
OG-00601, [1:33]Sql syntax error: expected end but '1-3' found

ALTER SYSTEM SET MES_CPU_INFO = 'grac-verify';
OG-00601, [1:33]Sql syntax error: expected end but 'grac-verify' found
```

## `MES_CRC_CHECK_SWITCH`

**参数说明**：设置 MES 消息的 CRC 校验开关。开启后对 MES 消息启用 CRC 校验；校验的覆盖范围属实现细节，**待确认**。

**取值范围**：布尔值，`FALSE` / `TRUE`（参数视图 `RANGE` 为 `FALSE,TRUE`）。

**默认值**：`TRUE`。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。实测：设为 FALSE → 重启 → 读回 `RUNTIME_VALUE=FALSE`；还原 TRUE → 重启 → 读回 TRUE/TRUE。

**配置建议**：关闭校验会失去 MES 消息的完整性校验能力。生产环境建议保持 `TRUE`；确需关闭时请评估通信环境的可信度，并在验证后恢复。

**示例**：

```sql
ALTER SYSTEM SET MES_CRC_CHECK_SWITCH = FALSE;   -- 持久化（默认 SCOPE=BOTH）
-- 重启后读回 FALSE / FALSE
ALTER SYSTEM SET MES_CRC_CHECK_SWITCH = TRUE;    -- 还原
-- 重启后读回 TRUE / TRUE
```

## `MES_POOL_SIZE`

**参数说明**：设置 MES 模块的内存池大小。该参数的具体消费方式（池数量或容量、单位换算）在 7.0.0-LTS 的公开文档与安装包中未见说明，**实现细节待确认**；本节仅描述参数配置接口层面的实测结果。

**取值范围**：整数，`[256,16384]`。

**默认值**：`256`。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。实测：设为 300 → 重启 → 读回 `RUNTIME_VALUE=300`；还原 256 → 重启 → 读回 256/256。

**配置建议**：调整会影响 MES 相关的内存占用，请结合实例总体内存余量评估；建议先小幅偏移验证，实例内存不足时可能导致启动失败，需预留回滚方案。

**示例**：

```sql
ALTER SYSTEM SET MES_POOL_SIZE = 300;   -- 持久化（默认 SCOPE=BOTH）
-- 重启后读回 300 / 300
ALTER SYSTEM SET MES_POOL_SIZE = 256;   -- 还原（下限值）
-- 重启后读回 256 / 256
```

## `MES_SSL_CRT_KEY_PATH`

**参数说明**：设置 MES 的 SSL 证书密钥路径。7.0.0-LTS 实测：**该参数要求路径指向已存在的目录**——指向已存在的目录设置成功；指向该目录下不存在的文件被拒绝。

**取值范围**：字符串路径；参数视图未提供范围。

**默认值**：（空）。单节点、未启用 SSL（`MES_SSL_SWITCH=FALSE`）的部署下为空。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。

**配置建议**：与 `MES_SSL_SWITCH`、`MES_SSL_KEY_PWD` 联动——**开启 SSL 前必须先配置有效的证书目录与密钥口令（口令须为 Base64 编码）**。7.0.0-LTS 实测：在未配置有效证书的情况下将 `MES_SSL_SWITCH` 持久化为 TRUE 并重启，ogracd 无法启动（详见 issue #62 的边界证据）；路径须为已存在目录，且 oGRAC 运行用户具备读权限。

**边界证据**：

```sql
ALTER SYSTEM SET MES_SSL_CRT_KEY_PATH = '<已存在目录>';
Succeed.

ALTER SYSTEM SET MES_SSL_CRT_KEY_PATH = '<已存在目录>/grac.crt';
OG-00601, [1:41]Sql syntax error: <已存在目录>/grac.crt is not an existing folder
```

## `MES_SSL_KEY_PWD`

**参数说明**：设置 MES 的 SSL 证书密钥口令。属敏感参数：**口令值不应写入文档、不应出现在任何公开截图或平台字段中**。7.0.0-LTS 实测：可通过 `SCOPE=MEMORY` 临时设置（不写入参数配置文件），设置返回 `Succeed.`，读回可见值——因含明文口令，不提供该参数的公开截图。

**取值须为 Base64 编码后的口令**：明文口令可被 `ALTER SYSTEM SET` 接受，但实例启动时解码失败；应填写对明文口令做 Base64 编码后的字符串。

**取值范围**：字符串；参数视图未提供范围。

**默认值**：（空）。单节点、未启用 SSL 的部署下为空。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。

**配置建议**：与 `MES_SSL_SWITCH`、`MES_SSL_CRT_KEY_PATH` 联动——开启 SSL 前必须先配置有效的证书目录与本口令。生产环境建议通过配置文件管理并控制文件权限，避免口令以明文形式出现在 SQL 历史或日志中；证书与口令须匹配同一套密钥。

**示例**（口令以占位符表示）：

```sql
-- 口令先做 Base64 编码（明文不要直接写进 SQL）
--   echo -n '<明文口令>' | base64
-- 仅用于临时验证：SCOPE=MEMORY 不写入参数配置文件，重启后失效
ALTER SYSTEM SET MES_SSL_KEY_PWD = '<BASE64 编码后的口令>' SCOPE=MEMORY;
```

## `MES_SSL_SWITCH`

**参数说明**：设置 MES 的 SSL 加密开关。开启后 MES 模块启用 SSL 加密传输。

**取值范围**：布尔值，`FALSE` / `TRUE`。

**默认值**：`FALSE`。

**生效方式**：重启生效（`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`）。

**重要边界（7.0.0-LTS 实测）**：在**未配置有效证书目录与密钥口令**的情况下，将本参数持久化为 `TRUE` 并重启，**ogracd 进程无法启动**（cms 正常启动，ogracd 未起来）；回滚配置文件中该参数为 `FALSE` 后重启恢复。该现象表明本开关与证书配置强耦合：**开启前必须先完成 `MES_SSL_CRT_KEY_PATH` 与 `MES_SSL_KEY_PWD` 的有效配置**。失败时的具体报错以引擎日志为准（属实现细节，待确认），本节只给出可复现的操作序列与结果。

**配置建议**：

- 单节点、未启用 SSL 的部署建议保持默认 `FALSE`。
- 确需开启时按"先配置证书目录与密钥口令 → 再置本参数为 TRUE → 重启验证"的顺序操作，并预留配置文件备份以便回滚。
- 不建议在持久化开启后首次重启前关闭已有连接；重启失败时应优先回滚本参数。

```sql
-- 可复现的操作序列（不建议在生产环境重放）
ALTER SYSTEM SET MES_SSL_SWITCH = TRUE;    -- 持久化（默认 SCOPE=BOTH）
-- deploy stop / deploy start 重启
-- 结果：cms start success，ogracd 未启动（实例不可用）
-- 回滚：配置文件中恢复 MES_SSL_SWITCH = FALSE → 重启 → 恢复
```

## `SESSIONS`

**参数说明**：设置数据库实例中并发会话（Session）的最大数量。实际运行时的并发会话数不得超过此值。

### 会话组成说明

- **系统预留会话**：
  - 固定 32 个：用于内部任务（如资源回收、Checkpoint 等）
  - `PARALLEL_MAX_THREADS` 个：用于 SQL 并行执行框架
- **EMERG 用户会话**：数量由 `SUPER_USER_RESERVED_SESSIONS` 参数控制
- **普通用户会话**：
  - 至少需配置 1 个，否则数据库无法启动
  - **未开启并行回放时**：  
    普通用户会话数 = `SESSIONS − 32 − PARALLEL_MAX_THREADS − SUPER_USER_RESERVED_SESSIONS`
  - **开启并行回放时（仅备机）**：  
    可用会话数 = `SESSIONS − 32 − PARALLEL_MAX_THREADS − SUPER_USER_RESERVED_SESSIONS − LOG_REPLAY_PROCESSES − REPLAY_PRELOAD_PROCESSES`  
    （主机行为与未开启并行回放时一致）

### 配置建议

- 一般建议设为 `OPTIMIZED_WORKER_THREADS × 1.2`
- 在 DN 节点上，建议 `SESSIONS < OPTIMIZED_WORKER_THREADS`

### 修改生效方式

- **立即生效**：新值 ≥ 初始值 `init_count` 且 ≤ `1.5 × init_count`
- **需重启生效**：新值 < `init_count` 或 > `1.5 × init_count`（但 ≤ 16320）  
  > 其中 `init_count` 为数据库启动时的 `SESSIONS` 值。系统默认预留 50% 的扩容空间用于动态调整，因此实时调整仅支持扩容，不支持缩容。

### 注意事项

1. 修改 `SUPER_USER_RESERVED_SESSIONS` 后需重启生效。
2. 调整 `SESSIONS` 前，请确保 `_MAX_RM_COUNT` 满足约束条件。
3. 若 `SESSIONS > 13600`，则 `_MAX_RM_COUNT` 必须为 64 的倍数、大于 `SESSIONS`，建议设为 16320。

**取值范围**：整数，  
下限 = `33 + PARALLEL_MAX_THREADS + SUPER_USER_RESERVED_SESSIONS`，  
上限 = 16320  
**默认值**：200  

## **XA_SUSPEND_TIMEOUT**

**参数描述**

设置 XA 事务分支挂起后的默认超时时间，单位为秒。挂起分支超过其保存的超时时间后，XA 资源回收检查会对其执行回滚并释放资源。该参数也用于未显式指定专用超时时间时，恢复（resume）挂起 XA 分支的等待。

XA 接口显式指定有效的专用超时时间时，优先使用接口提供的值。该参数不是普通 SQL 执行超时，也不是所有 XA 事务阶段的统一超时。

**取值范围**

- 整数，`[1, 3600]`，单位为秒；参数视图中表示为 `(0,3600]`。
- `0` 不表示无限等待，设置时会被拒绝。

**默认值**

`60`

**生效方式**

立即生效。新发起且采用默认超时的 XA 操作读取新值；已保存超时时间的事务分支不会因为修改该参数而统一重置其超时时间。

**配置建议**

- 根据事务管理器调度、网络延迟和业务允许的挂起时长设置，避免过小导致正常挂起分支被回滚。
- 过大的值会延长挂起事务占用资源的时间。调整前检查 `DV_GLOBAL_TRANSACTIONS`，并与事务管理器的超时策略保持一致。
- 使用 `SCOPE=MEMORY` 可验证当前实例中的修改；持久化方式参见[配置数据库系统](../database_administration_guide/basic_management_of_database_system/configuring_the_database_system.md)。

## **ARCH_CLEAN_IGNORE_BACKUP**

**参数描述**

控制自动清理归档日志时，是否忽略备份恢复点对可清理归档范围的限制。

- `TRUE`：检查归档是否可清理时，不再要求其日志点早于备份恢复点。
- `FALSE`：同时检查恢复点和备份恢复点，保留不满足备份恢复点条件的归档。

该开关不会跳过所有备份保护：正在进行的备份、分阶段备份所需归档以及本地恢复点等仍有独立检查。

**取值范围**

布尔型，`FALSE`、`TRUE`。

**默认值**

`TRUE`

**生效方式**

立即生效。自动清理是否执行，还取决于归档模式、`MAX_ARCH_FILES_SIZE`、清理上下限及归档是否满足清理条件。

**配置建议**

- 需要按备份恢复点保留归档时设置为 `FALSE`，并配合备份计划、归档保存策略和容量监控。
- 设置为 `TRUE` 前，应确认需要的备份恢复链已有可靠的归档保留措施；不能将它理解为无条件删除归档的开关。

## **ARCH_CLEAN_IGNORE_STANDBY**

**参数描述**

控制归档清理受备库恢复进度限制时，是否允许进一步按本地恢复点清理。

- `FALSE`：不启用忽略备库恢复点的额外清理步骤。
- `TRUE`：先按正常恢复点约束清理；若归档大小仍超过清理上限，且本地恢复点与考虑备库后的最小恢复点不同，则允许继续按本地恢复点清理，目标为清理上限对应的容量。

该开关不会绕过本地恢复点及其他仍然适用的归档保护条件，也不表示设置后立即执行一次清理。

**取值范围**

布尔型，`FALSE`、`TRUE`。

**默认值**

`FALSE`

**生效方式**

立即生效。

**配置建议**

- 正常主备部署保持 `FALSE`，为备库追赶保留需要的归档。
- 设置为 `TRUE` 可能使落后备库缺少追赶所需归档。仅在评估容量压力、备库恢复方案及补充归档或重建代价后使用。
- 结合 `MAX_ARCH_FILES_SIZE`、`ARCH_CLEAN_UPPER_LIMIT` 和实际备库延迟评估，不能仅依据参数设置成功判断备库恢复链是否完整。

## **ARCH_CLEAN_LOWER_LIMIT**

**参数描述**

设置自动归档清理的目标低水位，按 `MAX_ARCH_FILES_SIZE` 的百分比计算：

`目标容量 = MAX_ARCH_FILES_SIZE × ARCH_CLEAN_LOWER_LIMIT / 100`

触发自动清理后，系统尝试按归档文件粒度清理至目标容量以下。恢复、备份等保护条件可能使目标无法达到；该参数不表示必须精确保留指定字节数。

**取值范围**

- 整数，`[0, 100]`，单位为百分比。
- 必须满足 `ARCH_CLEAN_LOWER_LIMIT <= ARCH_CLEAN_UPPER_LIMIT`；设置时与当前运行中的上限比较，两个值可以相等。
- 设置为 `0` 也不会绕过归档的可清理性检查。

**默认值**

`30`

**生效方式**

立即生效。

**配置建议**

- 与上限配合形成适当间隔，平衡清理频率和归档保留量。调低下限会使一次清理尝试回收更多归档，调高则倾向于保留更多归档。
- 调整时始终保持下限不大于上限。下限不是整个文件系统的已用空间百分比。
- `MAX_ARCH_FILES_SIZE=0` 时，自动清理分支不执行；不能仅调整下限就期待发生清理。

## **ARCH_CLEAN_UPPER_LIMIT**

**参数描述**

设置触发自动归档清理的高水位，按 `MAX_ARCH_FILES_SIZE` 的百分比计算：

`触发容量 = MAX_ARCH_FILES_SIZE × ARCH_CLEAN_UPPER_LIMIT / 100`

当受管理的归档大小达到或超过该高水位，且满足自动清理的其他前提时，系统尝试清理归档。通常以 `ARCH_CLEAN_LOWER_LIMIT` 对应容量为目标，但仍须满足恢复、备份等保护条件，因此该高水位不是无条件保证的容量硬上限。

**取值范围**

- 整数，`[1, 100]`，单位为百分比。
- 必须满足 `ARCH_CLEAN_UPPER_LIMIT >= ARCH_CLEAN_LOWER_LIMIT`；设置时与当前运行中的下限比较。

**默认值**

`85`

**生效方式**

立即生效。

**配置建议**

- 调高上限会推迟达到清理触发条件，应预留足够磁盘空间；调低上限前，应评估备份和备库对现有归档的需求。
- 根据可用于归档的实际空间设置 `MAX_ARCH_FILES_SIZE`，同时为其他数据库文件和持续产生的日志预留容量。高水位不是文件系统整体使用率。
- `MAX_ARCH_FILES_SIZE=0` 时不执行该自动清理分支；归档模式未开启或归档受保护时，不能仅凭阈值推断清理结果。

## **ARCH_FILE_SIZE**

**参数描述**

设置 DBStor 归档路径中单个归档文件的大小阈值，用于归档写入及结束当前归档文件的判断。

该参数的作用与存储后端有关。普通文件归档路径根据 Redo 日志文件状态选择归档对象，不能仅凭该参数的值推断文件模式下一定按该大小切分归档文件。归档总容量另由 `MAX_ARCH_FILES_SIZE` 相关逻辑管理。

**取值范围**

- 整数大小，`[512M, 10G]`，基本单位为字节，可使用大小单位后缀。
- `M`、`G` 按 1024 进制计算，下限为 536870912 字节，上限为 10737418240 字节。

**默认值**

`10G`

**生效方式**

立即更新内存中的归档大小阈值，不会重写已有归档文件；具体归档行为由适用的存储路径在后续处理时使用该值。

**配置建议**

- 在适用的存储后端上，根据归档文件管理、备份传输和恢复需求选择大小，并结合 `ARCH_SIZE` 及归档时间策略评估。
- 试设可采用 `ALTER SYSTEM SET ARCH_FILE_SIZE='1G' SCOPE=MEMORY;`。设置前记录原值，验证后恢复；参数读回成功不等同于已经验证实际文件切分行为。

## **ARCH_SIZE**

**参数描述**

设置 DBStor 归档路径按待归档 Redo 数据量触发归档的大小阈值。存在待归档数据时，如果数据量尚未达到该值且归档时间阈值也未到，则继续等待；数据量或时间条件满足后可进入归档处理。

时间条件通常由 `ARCH_TIME` 控制，逻辑复制路径使用专用归档间隔。此外，仍须满足归档空间及内部检查间隔等条件。该参数不是归档目录总容量，也不是单个归档文件大小阈值。

**取值范围**

- 整数大小，`[0, 512M]`，基本单位为字节，可使用大小单位后缀。
- 参数视图的范围字符串为 `[0,512MB]`；内核校验的上限是 536870912 字节，即 512 × 1024 × 1024。
- `0` 不表示关闭归档：存在待归档数据时，大小阈值不再要求继续等待，但其他检查条件仍然适用。

**默认值**

`512M`

**生效方式**

立即更新内存中的归档触发阈值，由适用的归档路径在后续检查时使用。

**配置建议**

- 在适用的存储后端上，较小的值可能增加归档处理频次，应结合日志产生速率和归档时间策略调整。
- 试设可采用 `ALTER SYSTEM SET ARCH_SIZE='256M' SCOPE=MEMORY;`。测试时记录原值并恢复，不需要通过生成大量日志来验证设置是否被接受。
- 文件模式下应区分参数设置验证与 DBStor 归档触发功能验证；参数视图可读回新值并不证明执行了后者。

## **ARCHIVE_CONFIG**

**参数说明**：设置归档配置，取值为逗号分隔的组合值；当前配置为 `SEND,RECEIVE,NODG_CONFIG`。

**取值范围**：字符串（参数视图未提供范围）；默认值为 `SEND,RECEIVE,NODG_CONFIG`。

**默认值**：`SEND,RECEIVE,NODG_CONFIG`。

**生效方式**：**只读参数**（`MODIFIABLE=FALSE`、`EFFECTIVE=reboot`），不能通过 SQL 修改；调整须通过配置文件完成并重启。

```sql
ALTER SYSTEM SET ARCHIVE_CONFIG = 'SEND';
OG-00203, ARCHIVE_CONFIG is a readonly parameter
```

**配置建议**：只读参数，变更前请确认归档模式与业务影响，通过配置文件修改并安排在维护窗口重启；修改前备份归档配置与数据。

## **ARCHIVE_DEST_1**

**参数说明**：设置归档目标 1，取值格式为 `location=<目录>`。本次验证实例中该目标已由部署写入并被归档功能占用；**当归档已启用且该目标在用，修改本参数会被拒绝**（`OG-00857`，见边界证据）；未启用归档时的行为本次未验证。

**取值范围**：字符串，格式为 `location=<目录>`；参数视图未提供范围。

**默认值**：（空）；当前实例配置为 `location=<归档目录>`（随部署写入，以实际配置为准）。

**生效方式**：立即生效（`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`）。实测：归档功能正在使用该目标时，修改会被拒绝：

```sql
ALTER SYSTEM SET ARCHIVE_DEST_1 = 'location=/path/to/archive_dest_1';
OG-00857, ARCHIVE_DEST is in use, please disable it firstly
```

**配置建议**：修改在用的归档目标前，需先将该目标状态置为 `DEFER`（对应 `ARCHIVE_DEST_STATE_1` 参数，其文档在后续任务中补充），完成本参数修改后再恢复 `ENABLE`；否则会报 `OG-00857`。调整前备份归档数据。

## **ARCHIVE_DEST_2**

**参数说明**：设置归档目标 2，取值格式为 `location=<目录>`；当前未配置（空）。

**取值范围**：字符串，格式为 `location=<目录>`；参数视图未提供范围。

**默认值**：（空）。

**生效方式**：立即生效（`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`）。实测：带 `location=` 前缀设置成功，读回 `VALUE` 与 `RUNTIME_VALUE` 同步更新；**不带 `location=` 前缀的裸路径会被拒绝**，报错中的参数名为占位名 `ARCHIVE_DEST_n`：

```sql
ALTER SYSTEM SET ARCHIVE_DEST_2 = '/path/to/archive_dest_2';
OG-00202, The parameter value "ARCHIVE_DEST_n" was invalid
```

**配置建议**：

- 取值须为 `location=<目录>` 形式；目录应已存在并具备写权限。
- 临时验证可用 `SCOPE=MEMORY`（不写入配置文件），持久化请省略 scope。
- 参数视图未提供范围；单节点部署下归档目标的实际消费行为未验证，请在目标部署形态下确认后再用于生产。

**示例**：

```sql
-- 临时验证（不写入配置文件）
ALTER SYSTEM SET ARCHIVE_DEST_2 = 'location=/path/to/archive_dest_2' SCOPE=MEMORY;
SELECT NAME, DATATYPE, RANGE, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='ARCHIVE_DEST_2';
-- VALUE = RUNTIME_VALUE = location=/path/to/archive_dest_2

-- 置空（SCOPE=MEMORY 下可执行成功）
ALTER SYSTEM SET ARCHIVE_DEST_2 = '' SCOPE=MEMORY;
```

## **ARCHIVE_DEST_3**

**参数说明**：设置归档目标 3，取值格式为 `location=<目录>`；当前未配置（空）。

**取值范围**：字符串，格式为 `location=<目录>`；参数视图未提供范围。

**默认值**：（空）。

**生效方式**：立即生效（`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`）。取值格式与 `ARCHIVE_DEST_2` 一致：带 `location=` 前缀设置成功并立即读回；裸路径被拒绝（报错中的参数名为占位名 `ARCHIVE_DEST_n`）。

**配置建议**：同 `ARCHIVE_DEST_2`（目录须存在、注意写权限、临时验证用 `SCOPE=MEMORY`）。

**示例**：

```sql
ALTER SYSTEM SET ARCHIVE_DEST_3 = 'location=/path/to/archive_dest_3' SCOPE=MEMORY;
SELECT NAME, DATATYPE, RANGE, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='ARCHIVE_DEST_3';
-- VALUE = RUNTIME_VALUE = location=/path/to/archive_dest_3

ALTER SYSTEM SET ARCHIVE_DEST_3 = '' SCOPE=MEMORY;   -- 置空
```

## **ARCHIVE_DEST_10**

**参数说明**：设置归档目标 10，取值格式为 `location=<目录>`；当前未配置（空）。引擎注册了 `ARCHIVE_DEST_1` 至 `ARCHIVE_DEST_10` 共十个归档目标参数（master 源码 `srv_param.c:262-291`），编号连续；本 issue 的参数清单覆盖其中的 1、2、3、10 四个，其余编号不在本 issue 范围。

**取值范围**：字符串，格式为 `location=<目录>`；参数视图未提供范围。

**默认值**：（空）。

**生效方式**：立即生效（`MODIFIABLE=TRUE`、`EFFECTIVE=immediately`）。取值格式与 `ARCHIVE_DEST_2` 一致。

**配置建议**：同 `ARCHIVE_DEST_2`。`LOG_ARCHIVE_DEST_1/2/3/10` 是 `ARCHIVE_DEST_1/2/3/10` 的**别名**（同一配置项的两个名字，`config_item_t.alias`），因此取值始终一致；查询 `ARCHIVE_DEST_1` 时会同时带出 `ARCHIVE_DEST_10`，属名称前缀包含（两者是不同参数），不代表参数间联动。

**示例**：

```sql
ALTER SYSTEM SET ARCHIVE_DEST_10 = 'location=/path/to/archive_dest_10' SCOPE=MEMORY;
SELECT NAME, DATATYPE, RANGE, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='ARCHIVE_DEST_10';
-- VALUE = RUNTIME_VALUE = location=/path/to/archive_dest_10

ALTER SYSTEM SET ARCHIVE_DEST_10 = '' SCOPE=MEMORY;   -- 置空
```

## **RAFT_PEER_ADDRS**

**参数描述**
配置 RAFT 对端节点的通信地址列表。内核启动加载该值后，会把它连同本地地址、节点编号和对端编号传给 RAFT 库。

**类型与取值范围**
`OG_TYPE_VARCHAR`。参数视图的 `RANGE` 为 `-`，未给出完整格式约束。启用 RAFT 时本参数不能为空；内核启动时按分号分隔地址组、按英文逗号分隔组内地址，每个地址项必须包含冒号，并校验冒号前的 IP 地址。该段代码不校验冒号后的端口值，端口及节点映射仍需按 RAFT 库和实际部署要求确认。

单组地址示例：`10.0.0.2:19111,10.0.0.3:19111`。此处仅展示 `IP:端口` 与逗号分隔的写法；部署时应使用实际对端地址和端口。

**默认值**
空字符串。

**生效方式**
重启生效。

**依赖与配置建议**

- 仅在规划启用 `ENABLE_RAFT` 的集群中，与 `RAFT_PEER_IDS`、`RAFT_NODE_ID`、`RAFT_LOCAL_ADDR` 一起配置；对端编号、地址顺序和实际网络拓扑应相互对应。
- 本参数的 SQL 设置接口只执行通用字符串校验，不能用“设置成功”判断 RAFT 通信已经建立。修改前确认各节点配置和回退方案；未启用 RAFT 时保持默认值。

**配套参数**

| 参数 | 作用及启用 RAFT 时的要求 |
| --- | --- |
| `ENABLE_RAFT` | RAFT 功能开关，默认 `FALSE`；关闭时内核跳过 RAFT 专用参数的启动加载。启用前需完成节点及网络配置。 |
| `RAFT_NODE_ID` | 当前节点编号；启用 RAFT 时不能为空，须为不小于 `1` 的无符号整数，并与集群节点规划一致。 |
| `RAFT_LOCAL_ADDR` | 当前节点的 RAFT 通信地址；启用 RAFT 时不能为空，使用 `IP:端口` 格式。启动加载会检查冒号及其前的 IP 地址，端口仍需按部署方案确认。 |

## **RAFT_PEER_IDS**

**参数描述**
配置 RAFT 对端节点编号列表；内核把该值与对端地址一同传给 RAFT 库初始化集群通信。

**类型与取值范围**
`OG_TYPE_VARCHAR`。参数视图的 `RANGE` 为 `-`。启动时启用 RAFT 的情况下不允许为空；编号列表的进一步格式和拓扑约束取决于 RAFT 库，不能仅由此参数的 SQL 设置结果推断。

**默认值**
空字符串。

**生效方式**
重启生效。

**依赖与配置建议**
与 `RAFT_PEER_ADDRS`、`RAFT_NODE_ID` 及实际节点清单一起规划；未启用 RAFT 时保持默认值。

## **RAFT_PRIORITY_LEVEL**

**参数描述**
配置传递给 RAFT 库的节点优先级级别。具体选主行为由 RAFT 库和优先级策略决定。

**类型与取值范围**
`OG_TYPE_VARCHAR`，内容为 `0` 至 `16` 的十进制无符号整数。SQL 设置时按字符串形式填写，例如 `'1'`。

**默认值**
`0`。

**生效方式**
重启生效。

**依赖与配置建议**
仅在已规划的 RAFT 集群中与 `RAFT_PRIORITY_TYPE` 一起调整；取值被接受不代表已改变实际选主结果。

## **RAFT_PRIORITY_TYPE**

**参数描述**
配置传递给 RAFT 库的优先级策略类型。

**类型与取值范围**
`OG_TYPE_VARCHAR`；`External`、`Random`、`Static`、`AZFirst`。

**默认值**
`External`。

**生效方式**
重启生效。

**依赖与配置建议**
在 RAFT 集群内统一规划策略，并按具体 RAFT 库版本确认各策略语义；单节点或未启用 RAFT 的部署保持默认值。

## **RAFT_RAFT_ENTRY_CACHE_MEMORY_SIZE**

**参数描述**
配置传递给 RAFT 库的日志条目缓存内存大小。

**类型与取值范围**
参数视图显示 `OG_TYPE_INTEGER`、`[1,-)`；实际配置值支持容量单位，例如 `1G`。最小值为 1 字节，视图未给出固定上限；还需考虑主机可用内存和 RAFT 库约束。

**默认值**
`2G`。

**生效方式**
参数视图标记为重启生效。该参数也注册了内存通知回调，不能据此认定 RAFT 库已在运行时调整缓存；以 `VALUE`、`RUNTIME_VALUE` 和实际 RAFT 状态分别判断。

**依赖与配置建议**
仅在启用 RAFT 且评估日志负载、内存预算后调整；避免为缓存分配过多内存。变更前规划重启及回退窗口。

## **RAFT_START_MODE**

**参数描述**
配置 RAFT 模块初始化时传入的启动模式。

**类型与取值范围**
`OG_TYPE_INTEGER`，`[0,3]`。现有数据库参数视图没有解释各数值的完整含义；应按所用 RAFT 库和集群部署方案选择。

**默认值**
`0`。

**生效方式**
重启生效。

**依赖与配置建议**
本参数与 `ENABLE_RAFT` 及 RAFT 集群启动流程相关，普通单节点部署保持 `0`。未经集群方案确认，不通过改动此值尝试恢复或切换角色。

## **RAFT_TLS_DIR**

**参数描述**
指定传递给 RAFT 库的 TLS 材料目录。空值时内核不向 RAFT 库传入此目录。

**类型与取值范围**
`OG_TYPE_VARCHAR`，参数视图的 `RANGE` 为 `-`。设置非空值时，内核要求路径对应已存在、可读写的目录，并校验长度及特殊字符；目录通过校验不代表 TLS 握手成功。

**默认值**
空字符串。

**生效方式**
重启生效。

**依赖与配置建议**
仅在按 RAFT 安全部署方案准备好各节点所需材料和目录权限后设置；避免把目录内容或密钥写入命令输出、日志和截图。未启用 RAFT 时保持默认值。

## **RAFT_TOKEN_VERIFY**

**参数描述**
配置传递给 RAFT 库的令牌校验开关。

**类型与取值范围**
`OG_TYPE_VARCHAR`；设置校验接受大写 `TRUE` 或 `FALSE`。参数视图的 `RANGE` 为 `-`，不能据此推断可接受其他拼写。

**默认值**
`FALSE`。

**生效方式**
重启生效。

**依赖与配置建议**
启用前按 RAFT 库的安全配置要求准备令牌并协调集群各节点；未启用 RAFT 或尚未完成安全部署时保持默认值。设置接口接受 `TRUE` 不等于令牌校验已经工作。

## **RAFT_PENDING_CMDS_BUFFER_SIZE**

**参数描述**：设置 RAFT 待处理命令缓冲区大小。

**类型与取值范围**

`OG_TYPE_INTEGER`；取值范围 `[1, 4294967295]`（下限 1；上限按 32 位无符号整数校验）。

**默认值**：1000（7.0.0-LTS 实例 `DV_PARAMETERS.DEFAULT_VALUE`）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`。7.0.0-LTS 实测：`ALTER SYSTEM SET` 后 `VALUE` 立即变、`RUNTIME_VALUE` 不变；重启后 `RUNTIME_VALUE` 才等于新 `VALUE`。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议**：视图 `RANGE` 为 `[1,-)`，即未给出固定上限。7.0.0-LTS 实测下限被拒，报错原文：

```
alter system set RAFT_PENDING_CMDS_BUFFER_SIZE = 0;  ->  OG-00209, The value of parameter "RAFT_PENDING_CMDS_BUFFER_SIZE" is too small, at least 1
```

上限为 4294967295（32 位无符号整数校验上限）。修改需重启。**生效条件**：仅在启用 RAFT 的部署中生效；单节点、未启用 RAFT 时该值仅保存，不代表已生效。

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 2000）
ALTER SYSTEM SET RAFT_PENDING_CMDS_BUFFER_SIZE = 2000 SCOPE=MEMORY;

-- 读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='RAFT_PENDING_CMDS_BUFFER_SIZE';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 1000，请与实际实例核对后再执行）
ALTER SYSTEM SET RAFT_PENDING_CMDS_BUFFER_SIZE = 1000 SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

## **RAFT_RECEIVE_BUFFER_SIZE**

**参数描述**：设置 RAFT 接收缓冲区大小。

**类型与取值范围**：`OG_TYPE_INTEGER`；取值范围 `[1,10000]`。

**默认值**：100（7.0.0-LTS 实例 `DV_PARAMETERS.DEFAULT_VALUE`）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`。7.0.0-LTS 实测：`ALTER SYSTEM SET` 后 `VALUE` 立即变、`RUNTIME_VALUE` 不变；重启后 `RUNTIME_VALUE` 才等于新 `VALUE`。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议**：取值范围 `[1,10000]`。7.0.0-LTS 实测越界被拒，报错原文：

```
alter system set RAFT_RECEIVE_BUFFER_SIZE = 99999;  ->  OG-00211, The value of parameter "RAFT_RECEIVE_BUFFER_SIZE" should be in [1, 10000]
```

（`OG-00211` 是 7.0.0-LTS 实测见到的第三类越界错误码，`should be in [a, b]` 形式，与 `OG-00209`（too small）、`OG-00210`（too large）并列。）修改需重启。**生效条件**：仅在启用 RAFT 的部署中生效；单节点、未启用 RAFT 时该值仅保存，不代表已生效。

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 200）
ALTER SYSTEM SET RAFT_RECEIVE_BUFFER_SIZE = 200 SCOPE=MEMORY;

-- 读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='RAFT_RECEIVE_BUFFER_SIZE';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 100，请与实际实例核对后再执行）
ALTER SYSTEM SET RAFT_RECEIVE_BUFFER_SIZE = 100 SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

## **RAFT_SEND_BUFFER_SIZE**

**参数描述**：设置 RAFT 发送缓冲区大小。

**类型与取值范围**：`OG_TYPE_INTEGER`；取值范围 `[1,10000]`。

**默认值**：100（7.0.0-LTS 实例 `DV_PARAMETERS.DEFAULT_VALUE`）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`。7.0.0-LTS 实测：`ALTER SYSTEM SET` 后 `VALUE` 立即变、`RUNTIME_VALUE` 不变；重启后 `RUNTIME_VALUE` 才等于新 `VALUE`。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议**：取值范围 `[1,10000]`。7.0.0-LTS 实测越界被拒，报错原文：

```
alter system set RAFT_SEND_BUFFER_SIZE = 0;  ->  OG-00211, The value of parameter "RAFT_SEND_BUFFER_SIZE" should be in [1, 10000]
```

修改需重启。**生效条件**：仅在启用 RAFT 的部署中生效；单节点、未启用 RAFT 时该值仅保存，不代表已生效。

**试设与恢复示例**（下列设置值取自 7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 200）
ALTER SYSTEM SET RAFT_SEND_BUFFER_SIZE = 200 SCOPE=MEMORY;

-- 读回
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='RAFT_SEND_BUFFER_SIZE';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 100，请与实际实例核对后再执行）
ALTER SYSTEM SET RAFT_SEND_BUFFER_SIZE = 100 SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

## **USE_RBP**

**参数描述**  
RBP 总开关。开启后，数据库可与 RBPS 配合保存和读取恢复所需页面，用于缩短实例或节点故障后的恢复时间。

**版本说明**

本节 RBP 参数定义以 oGRAC 7.0.0 实现为依据。在 openEuler 20.03/aarch64 的 7.0.0 发行包中，`DV_PARAMETERS` 实测包含 `USE_RBP`、`RBP_PORT` 和 `RBP_RT_ANALYSIS`；此前测试的 7.0.0-RC3 发行包未检出这三项。使用其他发行包前，应先在实际实例的参数视图中核对可用项。

默认关闭：未部署 RBPS 或不需要恢复加速时，关闭该能力可以避免引入额外资源开销。

**取值范围**

- 布尔型。
- `TRUE`、`ON`、`1`：启用 RBP。
- `FALSE`、`OFF`、`0`：关闭 RBP。
- 建议统一使用 `TRUE` / `FALSE`，便于与参数视图对照。

**默认值**  
`FALSE`

**生效方式**  
重启生效。可用 `SCOPE=PFILE` 写入参数文件，按部署流程重启后检查运行值；`MODIFIABLE=TRUE` 不表示无需重启即可启用 RBP。

**配置建议**

- 未部署 RBPS 或只进行普通恢复时，保持默认值。
- 需要使用 RBPS 进行恢复加速时，所有相关数据库节点均应设置为 `TRUE`，并确认 RBPS 服务已部署且网络可达。

## **RBP_FOR_RECOVERY**

**参数描述**  
控制恢复阶段是否允许使用 RBP 加速。该参数需要与 `USE_RBP=TRUE` 同时成立；当 RBPS 可用且满足恢复条件时，数据库可使用 RBPS 中的页面缩短恢复时间。

默认开启：用户显式开启 RBP 后，通常希望恢复阶段直接使用加速能力；同时保留独立开关，便于临时关闭恢复加速。

**取值范围**

- `TRUE`：恢复阶段允许使用 RBP。
- `FALSE`：恢复阶段不使用 RBP。

**默认值**  
`TRUE`

**生效方式**  
重启生效。

**配置建议**

- 常规恢复加速场景保持默认值。
- 需要临时关闭 RBP 恢复加速时，可设置为 `FALSE` 后重启实例。

## **RBP_IP**

**参数描述**  
配置数据库连接 RBPS 的目标地址列表，最多可配置 8 个 IPv4 地址。

在两节点 DTC 部署中，数据库侧通常配置为连接对端 RBPS。RBP 通信建议与业务访问使用不同网卡。

**类型**

`OG_TYPE_VARCHAR`。

**取值范围**

- 非空字符串。
- 可配置英文逗号分隔的 IPv4 地址列表，最多 8 个地址。
- 地址列表不支持空项或分号分隔。

**默认值**  
`127.0.0.1`

**生效方式**  
重启生效。

**配置建议**

- 仅在计划使用 RBP/RBPS 时，根据 `USE_RBP`、`RBP_PORT` 及 RBPS 的实际部署配置目标地址；该地址列表不会自行启动 RBPS。
- 根据部署规划填写 RBPS 路由目标地址，例如 `node1_rbp_ip,node0_rbp_ip`；配置时需将示例占位符替换为实际 IPv4 地址。
- 按节点编号配置地址列表时，确保地址顺序与部署规划中的节点和 RBPS 路由关系一致。
- 如果 RBP 使用独立网卡，建议配置 RBP 网络上的地址，而不是业务访问地址。

## **RBP_PORT**

**参数描述**  
配置数据库连接 RBPS 的 TCP 端口。该端口必须与 RBPS 服务端 `rbps.conf` 中的监听端口一致。


**取值范围**  
整数，`[1024, 65535]`

该参数指定数据库连接 RBPS 的目标 TCP 端口，不是数据库 SQL 客户端监听端口。

**默认值**  
`2611`

**生效方式**  
重启生效。

**配置建议**

- 如需变更端口，需要同步修改所有数据库节点的 `RBP_PORT` 和所有相关 RBPS 的服务端监听端口，并重启对应组件。
- 写入参数文件后，应在重启及服务端端口调整完成后核对运行值和连通性；仅设置成功不能证明数据库已连接到新的 RBPS 端口。

## **LOCAL_RBP_HOST**

**参数描述**  
配置当前数据库实例上报给 RBPS 的本地地址。该参数用于 RBPS 侧日志、状态记录和读写阶段通知中识别数据库节点来源；它不决定数据库连接哪个 RBPS，连接目标由 `RBP_IP` 决定。

默认值为本机回环地址。

**取值范围**  
非空字符串。该参数仅支持配置单个 IPv4 地址，不支持主机名、逗号分隔列表或分号分隔列表。

**默认值**  
`127.0.0.1`

**生效方式**  
重启生效。

**配置建议**

- 集群部署中配置为当前数据库节点对 RBPS 可识别、可区分的 RBP 通信 IPv4 地址。
- 如果 RBP 使用独立网卡，建议配置独立网卡对应的地址。

## **RBP_RT_ANALYSIS**

**参数描述**  
控制是否开启 DTC RBP 运行期分析能力。开启后，数据库会在运行期间提前准备部分恢复所需信息，从而减少故障发生后的恢复准备时间。

默认关闭：运行期分析会引入额外资源开销，适合在明确需要缩短故障恢复时间的场景中按需开启。

**取值范围**

- 布尔型。
- `TRUE`、`ON`、`1`：开启运行期分析。
- `FALSE`、`OFF`、`0`：关闭运行期分析。

**默认值**  
`FALSE`

**生效方式**  
重启生效。

**配置建议**

- 该参数需要 `USE_RBP=TRUE`、`RBP_FOR_RECOVERY=TRUE` 才有实际意义。
- oGRAC 7.0.0 的运行期分析启用检查还要求 DTC 集群模式、节点数为 2，并且未启用 DBStor。单独将本参数设为 `TRUE` 不足以证明运行期分析线程已经工作。
- 对运行期资源消耗敏感，或不属于 DTC 恢复加速场景时，保持默认值。

## **RBP_RT_PARSE_WORKERS**

**参数描述**  
配置 DTC RBP 运行期分析的解析工作线程数量。

默认值为 `2` ：在两节点场景中，两个解析工作线程可以兼顾分析效率和 CPU 占用，避免初始部署时引入过多后台开销。

**取值范围**  
整数，`[1, 8]`

**默认值**  
`2`

**生效方式**  
重启生效。

**配置建议**

- 如果运行期分析处理速度不足，且 CPU 资源充足，可适当调大。
- 如果 CPU 压力较高或分析压力较小，可保持默认或调低。

## **RBP_RT_PAGE_OWNER_WORKERS**

**参数描述**  
配置 DTC RBP 运行期分析的页面归属处理工作线程数量。

默认值为 `4` ：页面归属处理通常需要比解析阶段更高的并发度，适当增加工作线程可以降低恢复信息处理积压风险。

**取值范围**  
整数，`[1, 8]`

**默认值**  
`4`

**生效方式**  
重启生效。

**配置建议**

- 一般保持默认值。
- 如果运行期分析处理积压明显，且内存和 CPU 资源充足，可适当调大。

## **ENABLE_PARA_LOG_FLUSH**

**参数描述**  
配置是否开启并行持久化REDO，需要在安装数据库时指定。开启后安装数据库时会自动根据当前主机环境的NUMA Cluster数量创建Redo文件。Redo文件数量等于（NUMA Cluster数量 * Redo Number * 2）。比如一个40个Cluster的主机，安装时设置Redo file number为3，如果安装双节点环境，需要40 * 3 * 2 = 240个Redo file。开启后会创建多个刷盘线程刷盘。相对地会占用较多的系统资源。

默认不开启。

**取值范围**

- `TRUE`：启用并行刷盘。
- `FALSE`：关闭并行刷盘。

**默认值**  
`FALSE`

**生效方式**  
安装时指定。

**配置建议**

- 一般保持默认值。

## **ENABLE_PARA_LOG_DFX**

**参数描述**  
配置是否开启并行持久化REDO运行时的详细运行日志打印，需要配合开启并行刷盘一起使用。打开后会在ogracd.rlog中打印更详细的运行日志，记录并行刷盘时redo写入和flush的详细信息，方便运维人员监控和进行问题分析。

默认不开启。

**取值范围**

- `TRUE`：启用并行刷盘详细日志打印。
- `FALSE`：关闭并行刷盘详细日志打印。

**默认值**  
`FALSE`

**生效方式**  
打开并行刷盘的前提下，修改后实时生效。

**配置建议**

- 一般保持默认值。

## **RBP_ASSEMBLE_MAX_SCAN**

**参数描述**  
配置 RBP 写入页面时单次最多扫描的候选项数量。适当放大该值可以提高页面批量发送效率，但也可能增加扫描开销。

默认值为 `300`。该值限制单次从 RBP 队列扫描的候选项数量；实际效率取决于队列内容和工作负载。

**类型**

`OG_TYPE_INTEGER`。

**取值范围**  
整数，`[100, 1000000]`

**默认值**  
`300`

**生效方式**  
立即生效。

**配置建议**

- 未启用 `USE_RBP` 或未部署 RBPS 时保持默认值；这时即使设置接口成功，也无法据此验证实际页面组装和发送行为。
- 在 RBP 写入队列确有扫描瓶颈时再调整；增大上限会允许单次遍历更多候选项，应结合负载与 CPU 开销评估。

## 数据库参数配置说明

### 1. 全局共享内存

#### `TEMP_BUFFER_SIZE`

**作用**：配置临时缓冲区（Temporary Buffer）的总大小，用于支持物化表创建、数据排序等临时操作。

- 系统在启动时会根据 `TEMP_BUFFER_SIZE`（总大小）和 `TEMP_POOL_NUM`（临时池数量）将缓冲区均分为多个临时池（Temp Pool）。
- 单个临时池的最小大小为 **2 GB**。若 `TEMP_BUFFER_SIZE < 2 GB`，则仅创建一个临时池。
- 支持**在线动态调大**：
  - 每次扩展的空间必须 ≥ `temp_buffer_extend_threshold`（临时缓冲区扩展阈值）。
  - 新增临时池数量 = `MIN(扩展值 / 单个临时池大小, 1)`。
  - 临时池总数上限为 **128**。
- **不支持在线调小**：若需减小该参数，需通过修改数据库配置文件或使用 `SCOPE=PFILE` 方式修改，并**重启数据库**生效。

**取值范围**：整数，[32M, 4T]（单位：Byte）  
**默认值**：32M  
**安装脚本默认值**：使用 `install.py` 安装时，自动设为 **1G**

**生效方式**

重启生效（7.0.0-LTS 实测 `EFFECTIVE=reboot`）。7.0.0-LTS 实测：`ALTER SYSTEM SET TEMP_BUFFER_SIZE = 1G` 返回 `Succeed.`，随后回退为 `2G`；实例启动加载的仍是 `2G`。

数据库启动时出现告警（原文来自7.0.0-LTS 实例运行日志 `ogracd.rlog`）：

```
The parameter temp pool num (8) is too large, reset to (1), each temp pool must not be smaller than (2147483648).
```

该告警说明的是配置文件中 `TEMP_POOL_NUM = 8` 因单个临时池不得小于 `2G` 而被重置为 `1`，**每次启动均会出现，并非设置 `1G` 所触发**。

#### `DATA_BUFFER_SIZE`

**作用**：设置数据缓冲区（Data Buffer）的大小，用于缓存最近访问的数据页，提升数据访问效率。

- 应根据服务器物理内存合理配置。
- **重启生效**，当前版本不支持在线调大或在线调小。执行 `ALTER SYSTEM SET DATA_BUFFER_SIZE = ...` 后，`SHOW PARAMETERS DATA_BUFFER_SIZE` 中的配置值 `VALUE` 可以更新，但当前进程使用的 `RUNTIME_VALUE` 不会改变，`EFFECTIVE` 显示为 `reboot`；重启数据库后才按新配置重新分配数据缓冲区。
- 参数值在配置阶段按取值范围校验，并在数据库启动时加载。规划调整时应同时评估物理内存以及其他共享内存参数的占用，确保数据库能够正常重启。

**取值范围**：整数，[64M, 32T]，支持单位：B / K / M / G / T（无单位时默认为字节）

**缺省值说明**：

  - 数据库内核未从配置文件读取到该参数时，使用 128M 作为回退值。该值不等同于安装后的实际初始值。
  - 随包配置模板 `pkg/cfg/ogracd.ini` 中预置为 2G。
  - 使用默认 LUN 部署配置时，内存自动调优默认开启，部署工具会根据物理内存计算该参数，当前算法将 4G 作为下限，因此安装后可能为 4G 或更大的值。
  - 其他部署方式或管理员显式配置可能覆盖上述值。该参数没有统一的安装后固定值，请以 `SHOW PARAMETERS DATA_BUFFER_SIZE` 的查询结果为准。

**配置建议**：

  - 修改前确认集群各节点的物理内存和配置一致性，并安排数据库重启窗口。
  - `SHOW PARAMETERS` 显示的具体值可能来自随包配置模板、部署工具或管理员后续配置。

#### `SHARED_POOL_SIZE`

**作用**：配置共享池（Shared Pool）的总大小，包含 Lock Pool、SQL Pool 和 DC Pool 等共享内存结构。

- **最小值受 `init_lock_pool_pages` 影响**：
  - 仅当 `init_lock_pool_pages = 128` 时，才允许将 `SHARED_POOL_SIZE` 设为最小值 **68M**。
- 备机的 `SHARED_POOL_SIZE` **必须 ≥ 主机值**，否则无法使用 68M 最小配置。
- 调大该参数后，建议评估是否需同步增大 `init_lock_pool_pages`。
- **动态调整限制**：
  - 仅支持**在线调大**，不支持在线收缩。
  - 单次扩展量（`expand_size`）必须 ≥ `MIN(512M, 当前 SHARED_POOL_SIZE)`，否则需重启生效。
  - 最多支持 **15 次**动态扩展；超过次数后，后续修改需重启数据库生效。

**取值范围**：整数，[68M, 32T]（单位：Byte）  
**默认值**：128M  
**配置建议**：

  - 如需调小，必须通过修改配置文件或使用 `SCOPE=PFILE`，并**重启数据库**生效。  
**安装脚本默认值**：使用 `install.py` 安装时，自动设为 **1G**

## **PMA_BUFFER_SIZE**

**参数描述**：设置 PMA 缓冲区大小。

**类型与取值范围**：`OG_TYPE_INTEGER`；取值范围 `[0,1T]`。

**默认值**：128M（7.0.0-LTS 实例 `DV_PARAMETERS.DEFAULT_VALUE`）。

**生效方式**：`MODIFIABLE=TRUE`、`EFFECTIVE=reboot`。7.0.0-LTS 实测：`ALTER SYSTEM SET` 后 `VALUE` 立即变、`RUNTIME_VALUE` 不变；`deploy stop` 后 `deploy start` 重启，重启后 `RUNTIME_VALUE` 才等于新 `VALUE`。使用 `SCOPE=MEMORY` 仅用于临时验证、不持久化；持久化请省略 scope（默认 `SCOPE=BOTH`）或使用 `SCOPE=PFILE`。

**配置建议**：视图 `RANGE` 为 `[0,1T]`。7.0.0-LTS 实测越界被拒，报错原文：

```
alter system set PMA_BUFFER_SIZE = 2T;  ->  OG-00601, [1:36]Sql syntax error: size value is bigger than maximum(1099511627776) required
```

1099511627776 字节 = 1T，与视图上限一致。修改需重启，请安排在维护窗口。

**试设与恢复示例**（下列设置值取自7.0.0-LTS 实测设置过的值）：

```sql
-- 试设（7.0.0-LTS 实测设置值 256M）
ALTER SYSTEM SET PMA_BUFFER_SIZE = 256M SCOPE=MEMORY;

-- 读回：VALUE 立即为新值，RUNTIME_VALUE 在重启前仍为旧值
SELECT NAME, VALUE, RUNTIME_VALUE FROM DV_PARAMETERS WHERE NAME='PMA_BUFFER_SIZE';

-- 恢复安装后原值（7.0.0-LTS 实例安装后原值为 128M，请与实际实例核对后再执行）
ALTER SYSTEM SET PMA_BUFFER_SIZE = 128M SCOPE=MEMORY;
-- 该参数为 reboot 生效：重启（deploy stop / deploy start）后 RUNTIME_VALUE 才恢复为新设置值
```

### 2. 检查点（Checkpoint）

#### `CHECKPOINT_IO_CAPACITY`

**作用**：控制每次 Checkpoint 触发时写入磁盘的脏页数量（以页为单位）。

**取值范围**：整数，[1, 4096]  
**默认值**：1024

**生效方式**

立即生效（7.0.0-LTS 实例 `DV_PARAMETERS` 实测 `EFFECTIVE=immediately`、`MODIFIABLE=TRUE`）。

7.0.0-LTS 实测 `ALTER SYSTEM SET CHECKPOINT_IO_CAPACITY = 5000 SCOPE=MEMORY;` 报错：

```
OG-00210, The value of parameter "CHECKPOINT_IO_CAPACITY" is too large, at most 4096
```

#### `_CHECKPOINT_TIMED_TASK_DELAY`

**作用**：设置两次定时 Checkpoint 任务之间的间隔时间（毫秒）。

- 定时 Checkpoint 由 `checkpoint_pages` 和 `checkpoint_period` 共同控制，本参数用于调节执行频率。

**取值范围**：整数，[0, 2³² − 1]（单位：毫秒）  
**默认值**：100

### 3. 事务类

#### `_UNDO_AUTO_SHRINK`

**作用**：控制是否启用 Undo Segment 的自动收缩功能。

**取值**：

- `true`：启用自动收缩
- `false`：禁用自动收缩

**默认值**：`true`

### 4. 进程类

#### `DBWR_PROCESSES`

**作用**：设置后台写脏页（DBWR）线程的数量。

- 增大该值可提升 I/O 并发能力，但会增加 CPU 和内存开销。
- **默认配置因安装包类型而异**：
  - 共享存储安装包：默认设为 **36**
  - 其他类型安装包：默认设为 **8**
- **特殊建议**：当 `FILE_OPTIONS = ASYNCH` 或 `SETALL`（即启用异步 I/O）时，建议将该参数设为 **1**，因为异步 I/O 本身已具备高并发能力，无需额外增加 DBWR 线程。

**取值范围**：整数，[1, 36]  
**默认值**：8  
**生效方式**：重启生效（EFFECT_REBOOT），修改后需按部署流程重启数据库实例。

### 5. 会话级参数

#### `_DOUBLEWRITE`

**作用**：控制是否启用双写（Double Write）机制，用于提升数据写入的可靠性，防止部分写（Partial Write）问题。

**取值**：

- `TRUE`：启用双写
- `FALSE`：关闭双写

**默认值**：`TRUE`

> **说明**：所有"仅支持在线调大"的参数，若需缩小，必须通过配置文件或 `SCOPE=PFILE` 修改，并**重启数据库实例**后生效。
