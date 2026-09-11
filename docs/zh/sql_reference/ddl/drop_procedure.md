# DROP PROCEDURE

## 功能描述

`DROP PROCEDURE` 语句用于删除数据库中已存在的存储过程。存储过程删除后，其名称和定义不再可用。

## 注意事项

- 默认可以删除自身 schema 下的存储过程。
- 删除其他用户 schema 下的存储过程，需要具备 `DROP ANY PROCEDURE` 系统权限。
- 对象名支持 `[schema_name.]procedure_name` 形式，省略 schema 时操作当前 schema 下的对象。
- 指定 `IF EXISTS` 时，目标存储过程不存在不会报错。

## 语法格式

```sql
DROP PROCEDURE [IF EXISTS] [schema_name.]procedure_name;
```

## 参数说明

- **procedure_name**: 要删除的存储过程名称。
- **schema_name**: 可选。存储过程所属的 schema 名称。省略时表示当前 schema。
- **IF EXISTS**: 可选。用于校验目标存储过程是否存在：指定该选项时，若目标不存在，语句直接返回执行成功；若目标存在，则执行删除操作。

## 示例

### 示例 1：删除单个存储过程

```sql
drop procedure if exists bison_ddl_proc;
```

### 示例 2：批量删除测试用存储过程

```sql
drop procedure if exists bison_ddl_proc_if;
drop procedure if exists bison_ddl_proc_modes;
drop procedure if exists bison_ddl_proc_args;
drop procedure if exists bison_ddl_proc_empty_args;
drop procedure if exists bison_ddl_proc;
```
