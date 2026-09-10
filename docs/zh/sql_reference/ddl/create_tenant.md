# CREATE TENANT

## 功能描述

`CREATE TENANT` 语句用于在 oGRAC 中创建租户，并指定该租户可用的表空间。租户是 oGRAC 的多租户隔离单位，租户内的用户以 `tenant$user` 形式命名。

> **说明：** 创建租户后，会话可通过 `ALTER SESSION SET TENANT = tenant_name;` 切换到指定租户。

## 注意事项

- 权限要求：执行 `CREATE TENANT` 语句的用户需具备 CREATE TENANT 系统权限。
- 命名限制：租户名长度上限为 32 个字符；租户名不得与已有租户重复，也不能为系统内置的 `TENANT$ROOT`。
- 数量限制：系统中租户数量上限为 256 个。
- 表空间要求：`TABLESPACES` 子句为必选，需至少指定一个表空间；列表中的每个表空间必须已存在且必须是用户表空间，否则报错 `OG-00780`（The tablespace does not exist.）。
- 表空间唯一性：同一个 `TABLESPACES` 列表中不能出现重复的表空间。
- 默认表空间：`DEFAULT TABLESPACE` 指定的表空间必须已存在，且必须已包含在 `TABLESPACES` 列表中；省略该子句时，默认使用 `TABLESPACES` 列表中的第一个表空间。

## 语法格式

```sql
CREATE TENANT tenant_name
    TABLESPACES ( tablespace_name [, ...] )
    [ DEFAULT TABLESPACE tablespace_name ];
```

## 参数说明

- **tenant_name**: 要创建的租户名称。长度不得超过 32 个字符，且不得与已有租户重名。

- **TABLESPACES ( tablespace_name [, ...] )**: 指定租户可用的表空间列表，可指定一个或多个表空间。每个表空间必须已存在且为用户表空间。

- **DEFAULT TABLESPACE tablespace_name**: 可选子句。指定租户的默认表空间，该表空间必须已存在且已包含在 `TABLESPACES` 列表中；省略时默认使用 `TABLESPACES` 列表中的第一个表空间。

## 示例

先创建租户所需的用户表空间，再创建租户：

```sql
-- 创建用户表空间
CREATE TABLESPACE bison_ddl_ts1
    DATAFILE '/home/ogracdba/data/bison_ddl_ts1.dbf' SIZE 100M;

-- 创建租户，指定可用表空间和默认表空间
CREATE TENANT bison_ddl_tenant
    TABLESPACES (bison_ddl_ts1)
    DEFAULT TABLESPACE bison_ddl_ts1;
```
