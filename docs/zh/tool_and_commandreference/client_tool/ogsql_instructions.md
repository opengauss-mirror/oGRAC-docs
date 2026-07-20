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
| Up/Down | 浏览上一条或下一条历史记录 |
| Ctrl+R | 在历史记录中进行反向搜索 |
| Tab | 根据当前输入上下文进行命令或数据库对象补全 |

反向搜索状态下，可以继续输入字符缩小匹配范围。重复按 Ctrl+R 可浏览下一条匹配记录，按 Enter 直接执行当前匹配，按 Esc 接受当前匹配并返回编辑状态，按 Ctrl+G 取消搜索并恢复搜索前的输入。

ogsql 根据终端上报的列数计算换行和光标位置。调整终端窗口大小后，下一次按键时会按新尺寸重新显示当前输入。不要通过 `stty cols` 设置与实际可见宽度不一致的列数，否则跨行移动和删除时可能出现显示错位。

### 历史记录

- 历史记录仅保存在当前 ogsql 进程的内存中，不写入磁盘；退出 ogsql 后记录自动清除。
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

在交互式编辑状态下按 Tab 可触发自动补全：

- 补全 SQL 关键字、ogsql 交互式命令和常用内置函数。
- 连接数据库后，根据 `FROM`、`JOIN`、`SELECT`、`CALL` 等上下文查询并补全表、视图、列、存储过程、函数和序列名称。
- 支持以 `schema.` 为前缀补全指定模式下的表和视图。
- 只有一个候选项时直接补全剩余字符；存在多个候选项时，先补全公共前缀，无法继续缩小时显示候选列表。
- 动态对象查询失败不会中断当前输入，也不会执行正在编辑的 SQL；ogsql 会在适用场景下回退到静态关键字补全。

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
