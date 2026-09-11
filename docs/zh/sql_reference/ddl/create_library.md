# CREATE LIBRARY

## 功能描述

注册一个共享库（.so 文件），供 CREATE FUNCTION ... AS LANGUAGE C 语句引用。库对象保存共享库文件的路径信息，是使用 C 语言编写的外部函数的基础。

## 注意事项

- 创建本 schema 下的库需要 CREATE LIBRARY 权限，创建其他 schema 下的库需要 CREATE ANY LIBRARY 权限
- 库路径为共享库文件在数据库节点上的路径，需用单引号括起来
- 支持 OR REPLACE 覆盖同名库，AS 与 IS 两种写法等价
- 库名支持 schema_name.library_name 形式，省略 schema 时为本用户 schema
- 删除库使用 DROP LIBRARY 语句

## 语法格式

```sql
CREATE [OR REPLACE] LIBRARY [schema_name.]library_name
    { AS | IS } 'library_path';
```

## 参数说明

- **OR REPLACE**: 可选，已存在同名库时替换
- **schema_name**: 库所属的 schema，省略时为本用户 schema
- **library_name**: 库名称
- **AS | IS**: 关键字，二者等价
- **'library_path'**: 共享库文件的路径，用单引号括起

## 示例

### 创建库

```sql
-- 创建共享库
CREATE LIBRARY lib_basic AS '/usr/lib/lib1.so';

-- 替换已存在的共享库
CREATE OR REPLACE LIBRARY lib_replace AS '/usr/lib/lib2.so';

-- 在 sys schema 下创建共享库
CREATE LIBRARY sys.lib_schema AS '/usr/lib/lib1.so';

-- 使用 IS 关键字创建共享库
CREATE LIBRARY lib_is IS '/usr/lib/lib1.so';
```

### 引用库创建 LANGUAGE C 函数

```sql
CREATE OR REPLACE LIBRARY bison_pl_c_noexists AS '/tmp/bison_pl_noexists.so';

CREATE OR REPLACE FUNCTION bison_pl_c_semicolon(p varchar2)
RETURN int
AS LANGUAGE C NAME og_ext_len LIBRARY bison_pl_c_noexists;
/
```

### 删除库

```sql
DROP LIBRARY IF EXISTS bison_pl_c_noexists;
```
