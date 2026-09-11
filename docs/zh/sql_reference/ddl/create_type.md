# CREATE TYPE

## 功能描述

CREATE TYPE 语句用于在当前 schema 或其他指定 schema 下创建自定义类型。oGRAC 支持对象类型、嵌套表、变长数组等自定义类型，类型创建后可被表列、变量、参数等引用。

## 注意事项

- 权限要求：在当前 schema 下创建自定义类型需要 **CREATE TYPE** 系统权限；在其他用户 schema 下创建需要 **CREATE ANY TYPE** 系统权限。
- `BODY` 关键字可选：不写 `BODY` 创建类型定义，写 `BODY` 创建类型体。
- `FORCE` 可选，指定后即使存在依赖也强制创建或替换。
- `FORCE` 可与 `OR REPLACE`、`IF NOT EXISTS` 组合使用。
- 类型体与类型定义可分别创建，两者使用同一个类型名。
- 对象类型的属性列表支持多个属性，属性之间以逗号分隔。
- `OR REPLACE` 与 `IF NOT EXISTS` 均用于处理同名对象已存在的情况：前者覆盖已有类型，后者在对象已存在时不报错、不创建。
- 对象名支持 `schema_name.type_name` 形式，也支持使用双引号包裹的引号名。
- `AS` 与 `IS` 两种写法等价，可任选其一。
- 类型定义必须以单独一行的 `/` 结束，否则语句无法正常执行。

## 语法格式

```sql
CREATE [OR REPLACE] TYPE [BODY] [IF NOT EXISTS] [schema_name.]type_name [FORCE]
    { AS | IS }
    ...类型定义...
/
```

## 参数说明

- **CREATE [OR REPLACE] TYPE**：创建类型。`OR REPLACE` 可选，指定后若同名类型已存在则覆盖重建。
- **BODY**：可选。指定该关键字时创建类型体；省略时创建类型定义。
- **IF NOT EXISTS**：可选。指定后若同名类型已存在，语句不报错，也不执行创建。
- **[schema_name.]type_name**：类型名称。`schema_name` 可选，省略时使用当前会话 schema；名称须符合对象命名规则，可使用双引号包裹以保留大小写或使用特殊字符。
- **FORCE**：可选。指定后即使存在依赖也强制创建或替换。
- **{ AS | IS }**：两种写法等价，用于引出类型定义。
- **类型定义**：常见形式如下。
  - **对象类型**：`AS OBJECT(attr type [, ...])`，属性列表由属性名和数据类型组成。
  - **嵌套表**：`IS TABLE OF element_type`，元素类型为 `element_type`。
  - **变长数组**：`IS VARRAY(n) OF element_type`，`n` 为数组的最大元素个数。
  - **类型体**：通过 `CREATE TYPE type_name BODY AS ...` 创建。
    - `UNDER parent_type`：用于指定父类型。
- **/**：语句结束符，必须单独占一行。

## 示例

创建对象类型：

```sql
create type bison_ddl_type as object(id int);
/
```

使用 `OR REPLACE` 和 `FORCE` 替换对象类型：

```sql
create or replace type bison_ddl_type force as object(id int, val int);
/
```

使用 `IF NOT EXISTS` 和 `FORCE` 创建对象类型：

```sql
create type if not exists bison_ddl_type_force force as object(id int, val int);
/
```

创建嵌套表类型：

```sql
create type bison_ddl_type_tab is table of int;
/
```

基于嵌套表类型创建变长数组类型：

```sql
create type bison_ddl_type_arr is varray(3) of bison_ddl_type_tab;
/
```
