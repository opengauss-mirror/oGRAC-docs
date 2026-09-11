# DROP TENANT

## 功能描述

`DROP TENANT` 语句用于删除数据库中的租户。删除租户时会同时删除该租户下的所有用户。

## 注意事项

- 权限要求：执行 `DROP TENANT` 语句的用户需具备 DROP TENANT 系统权限。
- 级联删除：删除租户时会同时删除该租户下的所有用户。
- 登录限制：若租户下有用户处于登录状态，删除操作失败并报错 `ERR_USER_HAS_LOGIN`（user has login）。
- `IF EXISTS`：租户不存在时不报错。

## 语法格式

```sql
DROP TENANT [IF EXISTS] tenant_name [CASCADE];
```

## 参数说明

- **IF EXISTS**: 可选参数。租户不存在时不报错。

- **tenant_name**: 要删除的租户名称。

- **CASCADE**: 可选参数，指定级联删除。

## 示例

先创建表空间和租户，再删除租户：

```sql
-- 创建用户表空间
CREATE TABLESPACE bison_ddl_ts1
    DATAFILE '/home/ogracdba/data/bison_ddl_ts1.dbf' SIZE 100M;

-- 创建租户
CREATE TENANT bison_ddl_tenant
    TABLESPACES (bison_ddl_ts1)
    DEFAULT TABLESPACE bison_ddl_ts1;

-- 删除租户及其下的所有用户
DROP TENANT IF EXISTS bison_ddl_tenant CASCADE;
```
