# ALTER TENANT

## 功能描述

`ALTER TENANT` 语句用于修改租户的表空间配置，包括为租户增加可用表空间，以及修改租户的默认表空间。语句提供两种相互独立的用法，一条语句只能执行其中一种操作。

## 注意事项

- 权限要求：执行 `ALTER TENANT` 语句的用户需具备 ALTER TENANT 系统权限。
- 租户必须已存在，否则报错 `OG-01394`（The tenant does not exist.）。
- `ADD TABLESPACES`：表空间必须已存在且为用户表空间；若该表空间已属于该租户，报错 `ERR_SPACE_ALREADY_USABLE`（The tablespace is already usable）。
- `DEFAULT TABLESPACE`：目标表空间必须已存在、为用户表空间，且必须已在该租户的可用表空间列表中，否则报错 `ERR_SPACE_DISABLED`；目标与当前默认表空间相同时，语句直接执行成功。
- 引用限制：租户被引用时不能修改。
- 两种用法相互独立，`ADD TABLESPACES` 与 `DEFAULT TABLESPACE` 不能在一条语句中同时使用。

## 语法格式

```sql
ALTER TENANT tenant_name ADD TABLESPACES ( tablespace_name [, ...] );

ALTER TENANT tenant_name DEFAULT TABLESPACE tablespace_name;
```

## 参数说明

- **tenant_name**: 要修改的租户名称，该租户必须已存在。

- **ADD TABLESPACES ( tablespace_name [, ...] )**: 为租户增加可用的表空间，可一次指定一个或多个表空间。表空间必须已存在且为用户表空间，且尚未加入该租户的可用表空间列表。

- **DEFAULT TABLESPACE tablespace_name**: 修改租户的默认表空间。目标表空间必须已存在、为用户表空间，且已在该租户的可用表空间列表中。

## 示例

先创建表空间和租户，再为租户增加表空间并修改默认表空间：

```sql
-- 创建用户表空间
CREATE TABLESPACE bison_ddl_ts1
    DATAFILE '/home/ogracdba/data/bison_ddl_ts1.dbf' SIZE 100M;

CREATE TABLESPACE bison_ddl_ts2
    DATAFILE '/home/ogracdba/data/bison_ddl_ts2.dbf' SIZE 100M;

-- 创建租户，先使用 ts1
CREATE TENANT bison_ddl_tenant
    TABLESPACES (bison_ddl_ts1)
    DEFAULT TABLESPACE bison_ddl_ts1;

-- 为租户增加可用表空间 ts2
ALTER TENANT bison_ddl_tenant ADD TABLESPACES (bison_ddl_ts2);

-- 将租户的默认表空间修改为 ts2
ALTER TENANT bison_ddl_tenant DEFAULT TABLESPACE bison_ddl_ts2;
```
