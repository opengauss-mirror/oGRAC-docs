# CREATE FUNCTION

## 功能描述

CREATE FUNCTION 语句用于在当前 schema 或其他指定 schema 下创建函数。函数是一段可以接收参数并返回单个值的 PL/SQL 程序单元，可在 SQL 表达式或其他 PL/SQL 程序单元中调用。

## 注意事项

- 权限要求：在当前 schema 下创建函数需要 **CREATE PROCEDURE** 系统权限；在其他用户 schema 下创建需要 **CREATE ANY PROCEDURE** 系统权限。
- 函数必须包含 `RETURN return_type` 子句，用于声明返回值类型。
- 参数语法与存储过程一致，支持 `IN`、`OUT`、`IN OUT` 三种模式，支持 `DEFAULT` 与 `:=` 两种默认值写法。
- `OR REPLACE` 与 `IF NOT EXISTS` 均用于处理同名对象已存在的情况：前者覆盖已有函数，后者在对象已存在时不报错、不创建。
- 对象名支持 `schema_name.function_name` 形式，也支持使用双引号包裹的引号名。
- `AS` 与 `IS` 两种写法等价，可任选其一。
- 函数体支持 `LANGUAGE C` 形式，用于绑定共享库中的外部函数；使用前需先通过 `CREATE LIBRARY` 注册共享库。
- 函数体必须以单独一行的 `/` 结束，否则语句无法正常执行。
- oGRAC 不支持 ALTER FUNCTION 语句。

## 语法格式

```sql
CREATE [OR REPLACE] FUNCTION [IF NOT EXISTS] [schema_name.]function_name
    [ ( { [argname] [argmode] argtype [ { DEFAULT | := } default_expr ] } [, ...] ) ]
    RETURN return_type
    { AS | IS }
    plsql_body
/
```

## 参数说明

- **CREATE [OR REPLACE] FUNCTION**：创建函数。`OR REPLACE` 可选，指定后若同名函数已存在则覆盖重建。
- **IF NOT EXISTS**：可选。指定后若同名函数已存在，语句不报错，也不执行创建。
- **[schema_name.]function_name**：函数名称。`schema_name` 可选，省略时使用当前会话 schema；名称须符合对象命名规则，可使用双引号包裹以保留大小写或使用特殊字符。
- **[ ( { [argname] [argmode] argtype [ { DEFAULT | := } default_expr ] } [, ...] ) ]**：参数列表，整体可选。
  - **argname**：参数名，可选。省略时只声明参数类型。
  - **argmode**：参数模式，可选，取值为 `IN`、`OUT`、`IN OUT`，默认 `IN`。
    - `IN`：输入参数，调用时传入值，函数体内只读。
    - `OUT`：输出参数，调用结束时向调用方返回该参数的值。
    - `IN OUT`：输入输出参数，调用时传入初值，函数体内可读写，并返回最终结果。
  - **argtype**：参数数据类型。
  - **{ DEFAULT | := } default_expr**：参数默认值，可选。`DEFAULT` 与 `:=` 等价，调用时未传入该参数则取默认值。
- **RETURN return_type**：必选。声明函数的返回值类型。
- **{ AS | IS }**：两种写法等价，用于引出函数体。
- **plsql_body**：PL/SQL 函数体，通常由可选的 `DECLARE` 声明区和 `BEGIN ... END;` 执行区组成，执行区中须包含 `RETURN` 语句。
- **LANGUAGE C**：函数体的外部实现形式，写法为 `AS LANGUAGE C NAME c_function_name LIBRARY library_name`，其中 `library_name` 为已通过 `CREATE LIBRARY` 注册的共享库。
- **/**：语句结束符，必须单独占一行。

## 示例

创建无参数的函数：

```sql
create function bison_ddl_func return int as
begin
    return 1;
end;
/
```

使用空参数括号创建函数：

```sql
create function bison_ddl_func_empty_args() return int is
begin
    return 1;
end;
/
```

创建带参数、带默认值的函数：

```sql
create or replace function bison_ddl_func_args(
    p_plain int,
    p_default int default 1,
    p_assign int := 2
) return int is
begin
    return p_plain;
end;
/
```

使用 `IF NOT EXISTS` 创建函数：

```sql
create function if not exists bison_ddl_func_if return int as
begin
    return 1;
end;
/
```

通过 `CREATE LIBRARY` 注册共享库后，创建 `LANGUAGE C` 形式的外部函数：

```sql
create or replace library bison_pl_c_noexists as '/tmp/bison_pl_noexists.so';
create or replace function bison_pl_c_semicolon(p varchar2)
return int
as language c name og_ext_len library bison_pl_c_noexists;
/
drop library if exists bison_pl_c_noexists;
```
