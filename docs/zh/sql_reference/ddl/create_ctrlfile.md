# CREATE CTRLFILE

## 功能描述

创建数据库控制文件。控制文件记录数据库的物理结构信息，包括重做日志文件、数据文件、字符集以及归档模式等，是数据库启动和恢复的关键文件。

## 注意事项

- 创建控制文件需要 CREATE CTRLFILE 权限
- 只能在 nomount 状态下创建控制文件
- 由 sys 用户在控制文件缺失的状态下创建
- 不允许重复创建控制文件
- LOGFILE、DATAFILE、CHARSET、ARCHIVELOG | NOARCHIVELOG 均为可选选项，可任意组合、可重复出现

## 语法格式

```sql
CREATE CTRLFILE
    [ LOGFILE ( 'file' [, ...] ) ]
    [ DATAFILE ( 'file' [, ...] ) ]
    [ CHARSET charset_name ]
    [ ARCHIVELOG | NOARCHIVELOG ];
```

## 参数说明

- **LOGFILE**: 指定重做日志文件，多个文件之间用逗号分隔，文件名用单引号括起
- **DATAFILE**: 指定数据文件，多个文件之间用逗号分隔，文件名用单引号括起
- **CHARSET**: 指定数据库字符集
- **ARCHIVELOG**: 指定数据库运行在归档模式
- **NOARCHIVELOG**: 指定数据库运行在非归档模式

## 示例

以下示例均需在 nomount 状态下由 sys 用户执行。

### 指定日志文件、数据文件、字符集和归档模式

```sql
CREATE CTRLFILE
    LOGFILE ('/home/ogracdba/ograc/redo01.log', '/home/ogracdba/ograc/redo02.log')
    DATAFILE ('/home/ogracdba/ograc/system01.dbf', '/home/ogracdba/ograc/user01.dbf')
    CHARSET UTF8
    NOARCHIVELOG;
```

### 指定数据文件并开启归档模式

```sql
CREATE CTRLFILE
    DATAFILE ('/home/ogracdba/ograc/system01.dbf')
    ARCHIVELOG;
```

### 指定日志文件

```sql
CREATE CTRLFILE
    LOGFILE ('/home/ogracdba/ograc/redo01.log');
```

### 只指定字符集

```sql
CREATE CTRLFILE
    CHARSET UTF8;
```
