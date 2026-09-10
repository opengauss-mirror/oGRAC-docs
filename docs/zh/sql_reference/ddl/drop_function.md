# DROP FUNCTION

## 功能描述

`DROP FUNCTION` 语句用于删除数据库中已存在的函数。函数删除后，其名称和定义不再可用。

## 注意事项

- 默认可以删除自身 schema 下的函数。
- 删除其他用户 schema 下的函数，需要具备 `DROP ANY PROCEDURE` 系统权限。
- 对象名支持 `[schema_name.]function_name` 形式，省略 schema 时操作当前 schema 下的对象。
- 指定 `IF EXISTS` 时，目标函数不存在不会报错。

## 语法格式

```sql
DROP FUNCTION [IF EXISTS] [schema_name.]function_name;
```

## 参数说明

- **function_name**: 要删除的函数名称。
- **schema_name**: 可选。函数所属的 schema 名称。省略时表示当前 schema。
- **IF EXISTS**: 可选。用于校验目标函数是否存在：指定该选项时，若目标不存在，语句直接返回执行成功；若目标存在，则执行删除操作。

## 示例

### 示例 1：删除单个函数

```sql
drop function if exists bison_ddl_func;
```

### 示例 2：批量删除测试用函数

```sql
drop function if exists bison_ddl_func_if;
drop function if exists bison_ddl_func_args;
drop function if exists bison_ddl_func_empty_args;
drop function if exists bison_ddl_func;
```
