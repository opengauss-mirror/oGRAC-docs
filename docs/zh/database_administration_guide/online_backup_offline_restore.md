# 在线备份与离线恢复

## 功能简介

`ogbackup`提供在线备份和离线恢复能力。在线备份阶段数据库保持运行，工具通过`ogsql`执行备份操作，将备份集写入指定目录；离线恢复阶段不启动`ogracd`和`ogsql`，直接根据本地备份仓中的备份集和恢复清单恢复物理文件。

离线恢复适用于需要在数据库服务未运行时恢复数据文件、控制文件、联机日志和归档日志的场景。恢复工具会校验备份链、备份分片、校验和以及压缩或加密的备份载荷；校验通过后才会开始改写目标文件。

> **说明：**
>
> 离线恢复只完成物理文件恢复，不执行日志应用、不执行`RECOVER DATABASE`、不执行`RESETLOGS`，也不会自动打开数据库。文件恢复成功后，需按照既有维护流程启动数据库，由内核完成恢复，并验证数据库能够正常打开。

## 前提条件

- 在数据库软件安装目录的`bin`子目录中执行`ogbackup`，或将该目录加入`PATH`环境变量。
- 在线备份使用的操作系统用户应具有连接数据库和写入备份目录的权限；备份目录必须为空或由本次备份专用。
- 离线恢复的`--backup-dir`必须是本地文件系统目录，目录中必须包含可用的备份集和`offline_restore.manifest`恢复清单。不能将DSS路径作为`--backup-dir`。
- 离线恢复前，必须停止所有`ogracd`和CMS Server进程。DSS存储场景还要求环境中的`DSS_HOME`可用，且只能保留一个当前用户启动的`dssserver`。
- 使用加密备份集时，恢复端必须提供创建备份时使用的密码。推荐通过权限为`0600`的普通密码文件传递，避免密码出现在命令行参数中。
- 恢复前应确认备份仓、目标数据目录和归档目录有足够空间，并先在测试环境验证恢复流程。

## 在线备份

### 备份目录

`--target-dir`指定备份父目录。工具会在该目录下创建`oGRAC`子目录，用于存放数据库生成的备份文件。

### 执行全量备份

未指定`--incremental`时，`ogbackup`执行级别0备份，即全量备份。建议在建立新的备份链、切换增量类型或完成一次恢复验证后执行全量备份。

```bash
ogbackup --backup --target-dir=/backup/ograc/full_20260901
```

带并行度和LZ4压缩的全量备份示例：

```bash
ogbackup --backup --target-dir=/backup/ograc/full_20260901 --parallel=4 --compress=lz4
```

`--parallel`取值范围为`1`到`16`。并行度应根据磁盘吞吐和数据库负载合理设置，避免备份任务影响在线业务。

### 执行增量备份

差异增量备份会以最近一次备份为基础，仅备份变化的数据；累积增量备份会以最近一次全量备份为基础，备份自该全量备份以来变化的数据。增量备份必须依赖完整的全量备份链，备份仓中不得删除其依赖的全量或中间备份集。

执行差异增量备份：

```bash
ogbackup --backup --target-dir=/backup/ograc/inc_20260902 --incremental
```

执行累积增量备份：

```bash
ogbackup --backup --target-dir=/backup/ograc/cum_20260903 --incremental --cumulative
```

> **说明：**
>
> 仅在新的全量备份之后才能切换差异增量和累积增量备份类型。

### 加密备份

使用`--password-file`可以为备份集设置加密密码。密码文件必须是非空普通文件，权限必须为`0600`，且不能与`--password`同时指定。

```bash
chmod 600 /backup/ograc/.backup_password
ogbackup --backup --target-dir=/backup/ograc/full_encrypt --password-file=/backup/ograc/.backup_password
```

### 备份完成后的处理

备份成功后，应保留本次备份目录中的全部文件，并将备份集、`offline_restore.manifest`和关联的全量或增量备份集作为一个整体复制或归档。删除备份链中的任意依赖备份集会导致离线恢复无法构建完整恢复链。

建议定期将备份仓复制到独立介质或异地存储，并通过离线恢复演练验证备份可用性。

## 离线恢复

### 恢复方式

离线恢复支持两种方式：

- 原地恢复：使用`--in-place`将数据文件、控制文件、联机日志和归档日志恢复到最终控制文件记录的原始路径。该操作会覆盖原有文件，必须同时指定`--force`确认。
- 目标目录恢复：不指定`--in-place`时，使用`--target-dir`指定恢复目标目录。目标目录必须使用绝对路径。DSS备份只支持正式的原地恢复模式。

离线恢复会自动选择恢复清单中最新的可用备份集；也可用`--backup-id`指定备份链末端，或使用`--target-time`选择完成时间不晚于指定时间戳的最新备份集。`--backup-id`和`--target-time`不能同时指定。

### 恢复前检查

执行实际恢复前，建议先使用`--dry-run`校验备份集和生成恢复计划。该模式不会创建、截断或写入目标文件。

原地恢复预检查示例：

```bash
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --in-place --force --dry-run
```

恢复到指定目录的预检查示例：

```bash
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --target-dir=/restore/ograc --dry-run
```

预检查会验证备份链的身份、版本、日志范围和分片连续性；对于加密或压缩备份集，还会验证解密和解压是否成功。预检查失败时，应先根据输出修复备份仓或环境问题，不要执行实际恢复。

### 执行原地恢复

确认预检查成功并已停止数据库相关进程后，执行原地恢复：

```bash
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --in-place --force
```

恢复指定备份链末端的示例：

```bash
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --backup-id=backup_20260903 --in-place --force
```

恢复加密备份集的示例：

```bash
chmod 600 /backup/ograc/.backup_password
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --in-place --force --password-file=/backup/ograc/.backup_password
```

> **注意：**
>
> `--in-place`不能与`--target-dir`同时使用。原地恢复会覆盖最终控制文件中记录的原始数据文件、控制文件、联机日志和归档日志，操作不可逆。执行前必须确认选定的备份链、恢复目标和归档路径正确。

### 执行目标目录恢复

将备份恢复到新的本地目录时，指定绝对路径的`--target-dir`。实际恢复会改写该目录中的目标文件，因此如目录已存在，需使用`--force`确认覆盖。

```bash
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --target-dir=/restore/ograc --force
```

离线恢复读取并串行处理备份分片，`--parallel`只允许取值`1`。

### 恢复完成后的操作

成功完成物理文件恢复后，备份仓中会生成`.ogbackup_offline_restore_file_phase_complete`标记文件。该标记表示文件阶段已完成，不表示数据库已经恢复并打开。

后续操作如下：

1. 按数据库维护流程启动相关服务和`ogracd`。
2. 让数据库内核完成恢复；如维护流程要求，执行相应的恢复、日志重置或打开操作。
3. 检查数据库状态为`OPEN`，并验证关键表、索引和应用数据。
4. 完成验证后，再恢复业务连接。

## 失败处理

恢复开始写入文件后，工具会在备份仓中维护故障保护标记：

- `.ogbackup_offline_restore_in_progress`：恢复正在进行，禁止启动该数据库。
- `.ogbackup_offline_restore_failed`：恢复在写入阶段失败，目标文件可能不一致，禁止启动该数据库。
- `.ogbackup_offline_restore_file_phase_complete`：物理文件恢复阶段完成，仍需执行后续数据库恢复和验证。

若存在`in_progress`或`failed`标记，必须先查明失败原因。确认需要重新开始恢复时，使用相同的恢复命令并增加`--force`，工具会重新准备所有计划目标并从头执行，不会跳过已处理文件或从中断位置续传。

```bash
ogbackup --offline-restore --backup-dir=/backup/ograc/repository --in-place --force
```

不要在存在失败标记的情况下直接启动数据库，也不要手工删除标记文件来绕过恢复检查。

## 参数说明

| 参数 | 适用操作 | 说明 |
| :--- | :--- | :--- |
| `--target-dir` | 在线备份、非原地离线恢复 | 在线备份时指定备份父目录；非原地恢复时指定绝对恢复目标目录。 |
| `--incremental` | 在线备份 | 执行级别1增量备份。 |
| `--cumulative` | 在线备份 | 指定增量备份为累积增量，必须与`--incremental`配合使用。 |
| `--parallel` | 在线备份、离线恢复 | 在线备份取值范围为`1`到`16`；离线恢复仅允许`1`。 |
| `--compress=lz4` | 在线备份 | 使用LZ4压缩备份集。 |
| `--password-file` | 在线备份、离线恢复 | 从权限为`0600`的普通文件读取备份加密密码。 |
| `--backup-dir` | 离线恢复 | 指定包含备份集和恢复清单的本地备份仓目录。 |
| `--backup-id` | 离线恢复 | 指定恢复链的末端备份集标识。 |
| `--target-time` | 离线恢复 | 选择完成时间不晚于该时间戳的最新备份集。 |
| `--dry-run` | 离线恢复 | 仅校验并打印恢复计划，不写入目标文件。 |
| `--in-place` | 离线恢复 | 按最终控制文件记录的原始路径恢复，必须与`--force`配合使用。 |
| `--force` | 离线恢复 | 确认覆盖目标文件，或在失败后重新开始恢复。 |
