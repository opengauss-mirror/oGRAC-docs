# CREATE PACKAGE

## 功能描述

CREATE PACKAGE 语句用于创建 PL/SQL 包。包是过程、函数、变量、游标、类型等程序单元的封装，由包规范（package specification）和包体（package body）两部分组成：包规范声明对外接口，包体提供具体实现。

## 注意事项

- 权限要求：在当前 schema 下创建包需要 **CREATE PROCEDURE** 系统权限；在其他用户 schema 下创建需要 **CREATE ANY PROCEDURE** 系统权限。
- `BODY` 关键字可选：不写 `BODY` 创建包规范，写 `BODY` 创建包体。
- 包规范中声明过程、函数、变量、游标、类型；包体中实现包规范里声明的过程与函数。
- `OR REPLACE` 与 `IF NOT EXISTS` 均用于处理同名对象已存在的情况：前者覆盖已有包，后者在对象已存在时不报错、不创建。
- 对象名支持 `schema_name.package_name` 形式，也支持使用双引号包裹的引号名。
- `AS` 与 `IS` 两种写法等价，可任选其一。
- `END` 后可省略包名，也可以重复书写包名。
- 包定义必须以单独一行的 `/` 结束，否则语句无法正常执行。

## 语法格式

```sql
CREATE [OR REPLACE] PACKAGE [BODY] [IF NOT EXISTS] [schema_name.]package_name
    { AS | IS }
    ...包声明或实现...
END [package_name];
/
```

## 参数说明

- **CREATE [OR REPLACE] PACKAGE**：创建包。`OR REPLACE` 可选，指定后若同名包已存在则覆盖重建。
- **BODY**：可选。指定该关键字时创建包体；省略时创建包规范。
  - 包规范：声明过程、函数、变量、游标、类型，只包含声明，不包含实现。
  - 包体：实现包规范中声明的过程与函数，可包含 PL/SQL 执行体。
- **IF NOT EXISTS**：可选。指定后若同名包已存在，语句不报错，也不执行创建。
- **[schema_name.]package_name**：包名称。`schema_name` 可选，省略时使用当前会话 schema；名称须符合对象命名规则，可使用双引号包裹以保留大小写或使用特殊字符。
- **{ AS | IS }**：两种写法等价，用于引出包规范或包体的内容。
- **包声明或实现**：包规范中书写过程、函数、变量、游标、类型等声明；包体中书写过程与函数的实现。
- **END [package_name]**：结束包定义。包名可选，重复书写时须与声明的包名一致。
- **/**：语句结束符，必须单独占一行。

## 示例

创建包规范：

```sql
create package bison_ddl_pkg as
    procedure p;
    function f return int;
end;
/
```

创建包体，实现包规范中声明的过程与函数：

```sql
create package body bison_ddl_pkg as
    procedure p as
    begin
        null;
    end;

    function f return int as
    begin
        return 1;
    end;
end;
/
```

使用 `OR REPLACE` 替换包规范：

```sql
create or replace package bison_ddl_pkg as
    procedure p;
    function f return int;
end;
/
```

使用 `OR REPLACE` 替换包体：

```sql
create or replace package body bison_ddl_pkg as
    procedure p as
    begin
        null;
    end;

    function f return int as
    begin
        return 1;
    end;
end;
/
```

使用 `IF NOT EXISTS` 创建包规范：

```sql
create package if not exists bison_ddl_pkg_if as
    procedure p;
end;
/
```

使用 `IF NOT EXISTS` 创建包体：

```sql
create package body if not exists bison_ddl_pkg_if as
    procedure p as
    begin
        null;
    end;
end;
/
```
