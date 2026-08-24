# ogsql使用说明

## 概述

ogsql 是 oGRAC 提供的 SQL 开发者命令行工具，用于连接数据库、执行 SQL 语句和脚本，以及管理数据库对象。本工具支持交互式和非交互式两种使用模式。

## 命令行语法

### 基本语法

ogsql 支持三种主要的使用方式：

```bash
# 方式1：显示帮助或版本信息
ogsql -h
ogsql -v

# 方式2：进入交互式模式
ogsql

# 方式3：指定登录信息和选项，可以以非交互模式执行 SQL 语句
ogsql [ <logon> [<options>] [<start>] ]
```

### 登录信息格式

ogsql 支持以下登录格式：

```
[ user [ /password ] @{host:port}[,...] ] [as sysdba]
```

以及特殊的管理员登录格式：

```
/ as { sysdba } [ host:port ]
```

其中各参数含义如下：

- **user**: 登录用户名
- **password**: 登录用户密码。如果不提供，将进入交互式输入模式（推荐出于安全考虑使用交互式输入）
- **host**: 数据库服务器IP地址，支持IPv4和IPv6
- **port**: 数据库服务器端口号
- **sysdba**: 数据库管理员身份

### 选项参数

ogsql 支持以下可选选项参数：

```
[-q] [-w <timeout>] [-a] [-D "data_home_path"]
```

各选项含义：

- **-q**: 取消 SSL 登录认证
- **-w \<timeout>**: 客户端连接数据库的超时时间（单位：秒）
  - 默认值为60秒
  - 特殊值-1表示无限超时
  - 特殊值0表示不等待
- **-a**: 打印已执行的 SQL 语句
  - 可与-f参数一起使用，表示打印并执行SQL脚本文件中的SQL语句
- **-D**: 指定数据目录路径

### 启动选项

ogsql 支持以下启动选项（同一时间只能使用其中一种）：

```
[-c "execute-sql-command"] | [-f "execute-sql-file"] | [-s "destination-file"]
```

各选项含义：

- **-c**: 执行指定的SQL语句
- **-f**: 执行指定的SQL脚本文件
- **-s**: 将命令提示符和输出重定向到指定文件

## 示例用法

### 基本登录

```bash
# 以管理员身份无密码登录
ogsql / as sysdba

# 通过IP地址和端口以指定用户登录
ogsql user/user_pwd@127.0.0.1:1611
```

### 执行SQL语句

```bash
# 登录后执行单个 SQL 语句
ogsql user/user_pwd@127.0.0.1:1611 -c "SELECT 1 FROM SYS_DUMMY"

# 执行SQL脚本文件
ogsql user/user_pwd@127.0.0.1:1611 -f "/home/user/example.sql"

# 打印并执行SQL脚本文件中的语句
ogsql user/user_pwd@127.0.0.1:1611 -a -f "/home/user/example.sql"
```

### 设置连接超时

```bash
# 设置连接超时为30秒
ogsql user/user_pwd@127.0.0.1:1611 -w 30

# 设置无限超时
ogsql user/user_pwd@127.0.0.1:1611 -w -1
```

## 交互式命令

在 ogsql 交互式模式下，可以使用以下命令：

| 命令 | 描述 | 示例 |
|------|------|------|
| CONN | 连接到数据库 | `CONN user/password@host:port` |
| EXIT/QUIT | 退出ogsql | `EXIT` |
| SHOW | 显示系统信息 | `SHOW PARAMETERS` |
| DESC | 描述数据库对象 | `DESC table_name` |
| @file | 执行SQL脚本文件 | `@/home/user/example.sql` |
| @@file | 执行嵌套SQL脚本文件 | `@@/home/user/include.sql` |
| SPOOL | 将输出保存到文件 | `SPOOL output.txt` |
| CLEAR | 清除屏幕 | `CLEAR` |
| SET | 设置ogsql环境变量 | `SET TIMING ON` |
| COLUMN | 设置列显示格式 | `COLUMN column_name FORMAT A20` |
| WHENEVER | 错误处理设置 | `WHENEVER SQLERROR EXIT` |
| PROMPT | 显示提示信息 | `PROMPT '开始执行脚本'` |

## 交互式输入编辑

### 生效条件

在 Linux 环境中，当 ogsql 以交互方式从标准输入读取命令，且标准输入和标准输出均连接到终端（TTY）时，默认启用交互式行编辑。通过管道重定向输入、使用 `-f` 执行脚本文件或在非 TTY 环境中运行时，ogsql 自动使用普通行输入方式，不启用历史浏览、反向搜索和按 Tab 补全。

交互式行编辑由 `HISTORY` 开关控制，默认开启。执行 `SET HISTORY OFF` 后，ogsql 从下一次输入开始切换为普通行输入；重新执行 `SET HISTORY ON` 后恢复交互式行编辑。

### 编辑快捷键

| 按键 | 功能 |
|------|------|
| Left/Right | 按字符向左或向右移动光标，支持 UTF-8 字符 |
| Home 或 Ctrl+A | 移动到当前输入开头 |
| End 或 Ctrl+E | 移动到当前输入末尾 |
| Backspace | 删除光标前的字符；当多行输入的当前续行为空时，回到上一逻辑行继续编辑 |
| Delete 或 Ctrl+D | 删除光标处的字符；输入为空时，Ctrl+D 结束当前 ogsql 会话 |
| Ctrl+U | 删除光标前的全部内容 |
| Ctrl+K | 删除光标后的全部内容 |
| Ctrl+W | 删除光标前的一个单词 |
| Ctrl+L | 清屏并重新显示当前输入 |
| Up/Down | 浏览上一条或下一条历史记录；在输入多行 SQL 时也可以浏览已保存的历史命令 |
| Ctrl+R | 在历史记录中进行反向搜索；输入多行 SQL 时不启用反向搜索 |
| Tab | 根据当前输入上下文进行命令或数据库对象补全 |

反向搜索状态下，可以继续输入字符缩小匹配范围。重复按 Ctrl+R 可浏览下一条匹配记录，按 Enter 直接执行当前匹配，按 Esc 接受当前匹配并返回编辑状态，按 Ctrl+G 取消搜索并恢复搜索前的输入。

在输入多行 SQL 的过程中按 Up 可以浏览已保存的历史命令。继续按 Down 返回到原来的多行输入，按 Esc 也可以恢复；在选中的历史命令上按 Enter，会放弃原来的多行输入并执行该命令。

ogsql 根据终端上报的列数计算换行和光标位置。调整终端窗口大小后，下一次按键时会按新尺寸重新显示当前输入。不要通过 `stty cols` 设置与实际可见宽度不一致的列数，否则跨行移动和删除时可能出现显示错位。

### 历史记录

- 历史记录按数据库用户名保存到客户端本地文件；使用同一操作系统用户和数据库用户重新登录后，可以通过 Up/Down 继续浏览之前的命令。不同数据库用户的历史记录相互隔离。
- 历史文件位于操作系统用户的主目录，文件名以 `.ogsql_history_` 开头，权限限制为当前操作系统用户可读写。无法获取主目录或写入文件时，不影响 ogsql 正常连接和执行命令。
- 默认保存最近 32 条完整命令，最多可配置为 100 条。多行 SQL 作为一条历史记录保存，并保留逻辑换行。
- 单条历史记录最多保存 4094 字节。超过限制的命令整条不进入历史，避免只保存不完整的 SQL。
- 包含密码、凭据、访问密钥或加解密关键字的命令不会进入历史。可以通过 `OGSQL_HISTORY_SENSITIVE_WORDS` 增加需要过滤的关键字。
- 执行 `SET HISTORY OFF` 后，关闭期间输入的命令不进入历史；重新开启后，关闭前已经保存的历史仍可浏览。

可以使用以下命令查看和设置历史开关，`HIST` 是 `HISTORY` 的缩写：

```sql
SHOW HISTORY;
SET HISTORY OFF;
SET HISTORY ON;
```

### 自动补全

在交互式编辑状态下按 Tab 可触发自动补全。多行 SQL 使用从语句开头到当前光标的完整逻辑输入判断上下文，行为与同一条单行 SQL 一致。终端使用 bracketed paste 序列传递粘贴内容时，其中的 Tab 作为空白字符插入，不触发自动补全。

支持的主要上下文如下：

| 输入位置 | 候选范围 |
|----------|----------|
| SQL 关键字位置 | SQL 保留字和受控补充关键字；在已识别的语法位置优先选择有效关键字，例如 `FROM`、`DISTINCT`、`SESSION`、`SYSTEM`、`IDENTIFIED`、`RESOURCE`、`REBUILD` |
| ogsql 一级命令 | ogsql 交互式命令 |
| 一级 `SET` 或 `SHOW` 后 | 当前版本实际支持设置或显示的客户端选项，例如 `HISTORY` |
| `FROM`、`JOIN`、`UPDATE`、`INSERT INTO`、`DESC` 等表对象位置 | 当前用户的表和视图；`FROM` 中以逗号分隔的后续表同样支持补全 |
| `ALTER INDEX`、`DROP INDEX` 后 | 当前用户的索引 |
| `ALTER SEQUENCE`、`DROP SEQUENCE` 后 | 当前用户的序列 |
| `CALL`、`EXEC`、`EXECUTE` 后 | 当前用户的存储过程和函数 |
| SELECT 等表达式位置 | 常用内置函数、当前用户的存储函数和序列；内置函数包括 `GS_ENCRYPT`、`GS_DECRYPT`、`GS_ENCRYPT_AES128`、`GS_DECRYPT_AES128` |
| `WHERE`、`ON`、`UPDATE ... SET` 等列位置 | 仅补全当前语句中已解析的 `FROM`、`JOIN`、`UPDATE` 或 `INSERT INTO` 来源表的列；尚未解析到来源表时不查询当前用户的全局列集合 |
| `表名.` 或 `别名.` 后 | 仅补全该表或别名对应的列 |
| `INSERT INTO 表名 (` 的未闭合列清单中 | 仅补全 INSERT 目标表的列 |
| 表对象位置的 `schema.` 后 | 当前连接用户可见的指定模式表和视图，包括通过对象权限可见的对象 |
| `CREATE TABLE` 的未闭合列定义中 | 受控类型和属性词，包括 `TEXT`、`UNSIGNED`、`AUTO_INCREMENT` |

候选处理规则和限制如下：

- 只有一个候选项时直接补全剩余字符；存在多个候选项时，先补全公共前缀，无法继续缩小时显示候选列表。
- 表、视图、索引、序列、存储过程和函数等明确对象位置，以及来源表列、INSERT 列清单和 `schema.` 后，支持在尚未输入名称前缀时列出候选；普通 SQL 关键字、内置函数、CREATE TABLE 类型或属性词和未识别语境需要至少输入一个前缀字符。
- 动态对象查询先根据对象类型、模式、来源表和已输入前缀过滤并稳定排序，再应用记录数限制。`OGSQL_COMPLETION_MAX_RECORDS` 控制单次动态查询最多读取的记录数，客户端最终最多保留 256 个候选。
- 一条 SQL 最多解析前 8 个不重复的顶层 `FROM`、`JOIN`、`UPDATE` 或 `INSERT INTO` 来源对象用于列补全；超出该范围的来源不参与列候选查询。
- 动态对象范围遵循当前连接用户的对象可见性和权限；对象不存在或当前用户不可见时不会出现在候选中。
- `ALTER INDEX ... REBUILD` 支持按已输入前缀补全 `REBUILD`。`LOCAL`、`GLOBAL` 等词可作为通用关键字候选出现，但补全器不校验其在具体 CREATE INDEX 位置是否符合语法。
- 当前不提供 `IF EXISTS` 短语级补全，也不把纯数字输入扩展为常量候选。
- 补全器只识别上述常用 SQL 结构，不是完整 SQL 语法解析器。复杂子查询、带双引号且区分大小写的对象名、数据库链接、同义词解析、PL/SQL 内部作用域和其他未列出的语法位置不保证对象级补全。
- 关键字出现在候选中只表示 ogsql 可以补出该词，不表示该关键字在任意 SQL 位置都合法；SQL 是否有效仍由数据库语法检查决定。
- 动态对象查询失败不会中断当前输入，也不会执行正在编辑的 SQL；ogsql 会保留适用的静态关键字或函数候选。
- ogsql 的自动补全范围独立于其他命令行客户端；不能根据其他客户端是否支持某个位置推断 ogsql 的行为。

## 参数绑定

ogsql 支持在 SQL 语句中使用参数绑定，格式为 `:参数名`。执行包含参数的 SQL 语句时，ogsql 会提示用户输入参数值。

示例：

```sql
SELECT * FROM employees WHERE department_id = :dept_id;
```

执行时，ogsql 会提示用户输入 dept_id 的值。

## 环境变量设置

使用 SET 命令可以配置 ogsql 的运行环境：

```sql
-- 启用执行时间统计
SET TIMING ON;

-- 启用SQL语句显示
SET ECHO ON;

-- 关闭或开启交互式行编辑和内存历史
SET HISTORY OFF;
SET HISTORY ON;
```

交互式行编辑还支持以下进程环境变量。环境变量在进程第一次进入交互式行编辑前读取，建议在启动 ogsql 前设置。

| 环境变量 | 默认值 | 取值范围或格式 | 说明 |
|----------|--------|----------------|------|
| `OGSQL_HISTSIZE` | `32` | `1`～`100` | 当前会话最多保留的历史记录条数 |
| `OGSQL_MAX_SQL_LEN` | `65536` | `1024`～`65536` | 交互式编辑允许输入的最大 SQL 字节数 |
| `OGSQL_COMPLETION_MAX_RECORDS` | `1024` | `1`～`10000` | 单次动态对象补全查询最多读取的记录数 |
| `OGSQL_HISTORY_SENSITIVE_WORDS` | 空 | 逗号分隔的关键字列表 | 增加不允许写入历史的敏感关键字，最多 64 个，每个关键字少于 128 字节 |

示例：

```bash
export OGSQL_HISTSIZE=50
export OGSQL_MAX_SQL_LEN=32768
export OGSQL_COMPLETION_MAX_RECORDS=2000
export OGSQL_HISTORY_SENSITIVE_WORDS="TOKEN,PRIVATE_KEY"
ogsql
```

对于无效的数值配置，ogsql 会输出警告并使用默认值；超过允许上限的数值会被限制为对应上限。

## 输出重定向

使用 SPOOL 命令可以将 ogsql 的输出重定向到文件：

```sql
-- 开始将输出保存到文件
SPOOL output.txt;

-- 执行SQL语句
SELECT * FROM employees;

-- 停止输出重定向
SPOOL OFF;
```

## 错误处理

使用 WHENEVER 命令可以设置 ogsql 在遇到错误时的行为：

```sql
-- 遇到SQL错误时退出
WHENEVER SQLERROR EXIT;

-- 遇到SQL错误时继续执行但显示错误
WHENEVER SQLERROR CONTINUE;
```

## 元数据查询示例

以下是一些常用的元数据查询示例：

```sql
-- 查看所有用户
SELECT * FROM ADM_USERS;

-- 查看所有表
SELECT * FROM ADM_TABLES;

-- 查看表结构
DESC table_name;

-- 查看系统参数
SHOW PARAMETERS;
```
