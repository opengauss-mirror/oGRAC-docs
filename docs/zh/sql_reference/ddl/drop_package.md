# DROP PACKAGE

## 功能描述

`DROP PACKAGE` 语句用于删除数据库中已存在的包（Package）。包由包规范（Package Specification）和包体（Package Body）两部分组成，该语句既可以删除整个包，也可以仅删除包体。

## 注意事项

- 不带 `BODY`：删除整个包，包括包规范及其包体。
- 带 `BODY`：仅删除包体，包规范保留。
- 默认可以删除自身 schema 下的包。
- 删除其他用户 schema 下的包，需要具备 `DROP ANY PROCEDURE` 系统权限。
- 指定 `IF EXISTS` 时，目标包不存在不会报错。

## 语法格式

```sql
DROP PACKAGE [IF EXISTS] [schema_name.]package_name;
DROP PACKAGE BODY [IF EXISTS] [schema_name.]package_name;
```

## 参数说明

- **package_name**: 要删除的包名称。
- **schema_name**: 可选。包所属的 schema 名称。省略时表示当前 schema。
- **IF EXISTS**: 可选。用于校验目标包是否存在：指定该选项时，若目标不存在，语句直接返回执行成功；若目标存在，则执行删除操作。
- **BODY**: 可选。指定时仅删除包体，保留包规范；不指定时删除整个包（包规范及其包体）。

## 示例

### 示例 1：仅删除包体

```sql
drop package body if exists bison_ddl_pkg_if;
```

### 示例 2：删除整个包

```sql
drop package if exists bison_ddl_pkg_if;
```
