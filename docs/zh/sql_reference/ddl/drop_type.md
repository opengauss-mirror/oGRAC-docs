# DROP TYPE

## 功能描述

`DROP TYPE` 语句用于删除数据库中已存在的自定义类型。类型由类型定义和类型体（Type Body）两部分组成，该语句既可以删除整个类型，也可以仅删除类型体。

## 注意事项

- 不带 `BODY`：删除整个类型。
- 带 `BODY`：仅删除类型体，类型定义保留。
- `FORCE`：当类型被其他类型引用时，需要指定 `FORCE` 才能删除；若类型被表使用，则不允许删除。
- 默认可以删除自身 schema 下的类型。
- 删除其他用户 schema 下的类型，需要具备 `DROP ANY TYPE` 系统权限。
- 指定 `IF EXISTS` 时，目标类型不存在不会报错。

## 语法格式

```sql
DROP TYPE [IF EXISTS] [schema_name.]type_name [FORCE];
DROP TYPE BODY [IF EXISTS] [schema_name.]type_name [FORCE];
```

## 参数说明

- **type_name**: 要删除的类型名称。
- **schema_name**: 可选。类型所属的 schema 名称。省略时表示当前 schema。
- **IF EXISTS**: 可选。用于校验目标类型是否存在：指定该选项时，若目标不存在，语句直接返回执行成功；若目标存在，则执行删除操作。
- **BODY**: 可选。指定时仅删除类型体，保留类型定义；不指定时删除整个类型。
- **FORCE**: 可选。当类型被其他类型引用时，指定该选项可强制删除；类型被表使用时无法删除。

## 示例

### 示例 1：删除单个类型

```sql
drop type if exists bison_ddl_type force;
```

### 示例 2：删除存在依赖的类型

```sql
drop type if exists bison_ddl_type_arr force;
drop type if exists bison_ddl_type_tab force;
drop type if exists bison_ddl_type_force force;
```
