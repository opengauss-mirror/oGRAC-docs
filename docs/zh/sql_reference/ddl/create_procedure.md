# CREATE PROCEDURE

## 功能描述

CREATE PROCEDURE 语句用于在当前 schema 或其他指定 schema 下创建存储过程。存储过程是一组预先编译的 PL/SQL 语句集合，可以声明输入、输出参数，供应用程序或其他 PL/SQL 程序单元调用。

## 注意事项

- 权限要求：在当前 schema 下创建存储过程需要 **CREATE PROCEDURE** 系统权限；在其他用户 schema 下创建需要 **CREATE ANY PROCEDURE** 系统权限。
- `OR REPLACE` 与 `IF NOT EXISTS` 均用于处理同名对象已存在的情况：前者覆盖已有存储过程，后者在对象已存在时不报错、不创建。
- 权限指定仅支持 `AUTHID CURRENT_USER`，指定后存储过程以调用者权限执行。
- 参数模式支持 `IN`、`OUT`、`IN OUT` 三种，省略模式时默认为 `IN`。
- 参数默认值支持 `DEFAULT` 与 `:=` 两种写法，二者等价。
- 参数列表整体可选；不声明参数时可以不写参数括号，也可以写成空括号 `()`。
- 对象名支持 `schema_name.procedure_name` 形式，也支持使用双引号包裹的引号名。
- `AS` 与 `IS` 两种写法等价，可任选其一。
- 存储过程体必须以单独一行的 `/` 结束，否则语句无法正常执行。

## 语法格式

```sql
CREATE [OR REPLACE] PROCEDURE [IF NOT EXISTS] [schema_name.]procedure_name
    [ ( { [argname] [argmode] argtype [ { DEFAULT | := } default_expr ] } [, ...] ) ]
    [ AUTHID CURRENT_USER ]
    { AS | IS }
    plsql_body
/
```

## 参数说明

- **CREATE [OR REPLACE] PROCEDURE**：创建存储过程。`OR REPLACE` 可选，指定后若同名存储过程已存在则覆盖重建。
- **IF NOT EXISTS**：可选。指定后若同名存储过程已存在，语句不报错，也不执行创建。
- **[schema_name.]procedure_name**：存储过程名称。`schema_name` 可选，省略时使用当前会话 schema；名称须符合对象命名规则，可使用双引号包裹以保留大小写或使用特殊字符。
- **[ ( { [argname] [argmode] argtype [ { DEFAULT | := } default_expr ] } [, ...] ) ]**：参数列表，整体可选。
  - **argname**：参数名，可选。省略时只声明参数类型。
  - **argmode**：参数模式，可选，取值为 `IN`、`OUT`、`IN OUT`，默认 `IN`。
    - `IN`：输入参数，调用时传入值，过程体内只读。
    - `OUT`：输出参数，调用结束时向调用方返回该参数的值。
    - `IN OUT`：输入输出参数，调用时传入初值，过程体内可读写，并返回最终结果。
  - **argtype**：参数数据类型。
  - **{ DEFAULT | := } default_expr**：参数默认值，可选。`DEFAULT` 与 `:=` 等价，调用时未传入该参数则取默认值。
- **AUTHID CURRENT_USER**：可选。指定存储过程以调用者权限执行，即过程体内 SQL 的权限校验按调用者身份进行。
- **{ AS | IS }**：两种写法等价，用于引出存储过程体。
- **plsql_body**：PL/SQL 过程体，通常由可选的 `DECLARE` 声明区和 `BEGIN ... END;` 执行区组成。
- **/**：语句结束符，必须单独占一行。

## 示例

创建无参数的存储过程：

```sql
create procedure bison_ddl_proc as
begin
    null;
end;
/
```

使用空参数括号创建存储过程：

```sql
create procedure bison_ddl_proc_empty_args() as
begin
    null;
end;
/
```

创建带参数、带默认值并指定 `AUTHID CURRENT_USER` 的存储过程：

```sql
create or replace procedure bison_ddl_proc_args(
    p_plain int,
    p_default int default 1,
    p_assign int := 2
) authid current_user is
begin
    null;
end;
/
```

创建包含 `IN`、`OUT`、`IN OUT` 三种参数模式的存储过程：

```sql
create procedure bison_ddl_proc_modes(
    p_in in int,
    p_out out int,
    p_io in out int
) is
begin
    p_out := p_in;
    p_io := p_in;
end;
/
```

使用 `IF NOT EXISTS` 创建存储过程：

```sql
create procedure if not exists bison_ddl_proc_if as
begin
    null;
end;
/
```
