# DROP LIBRARY

## 功能描述

DROP LIBRARY 语句用于删除数据库中已注册的库对象。库对象由 CREATE LIBRARY 语句创建，保存共享库文件的路径信息，供 CREATE FUNCTION ... AS LANGUAGE C 语句引用。删除库对象后，该库的名称可以被重新使用。

## 注意事项

- 默认可删除自身 schema 下的库；删除其他 schema 下的库需要 DROP ANY LIBRARY 权限
- IF EXISTS 选项在库不存在时不报错；不指定该选项时，删除不存在的库会报错
- 库名支持 schema_name.library_name 形式，省略 schema 时为本用户 schema

## 语法格式

```sql
DROP LIBRARY [IF EXISTS] [schema_name.]library_name;
```

## 参数说明

- **IF EXISTS**: 可选，库不存在时不报错
- **schema_name**: 库所属的 schema，省略时为本用户 schema
- **library_name**: 要删除的库名称

## 示例

以下示例展示 DROP LIBRARY 的常见用法。

### 删除本 schema 下的库

```sql
DROP LIBRARY lib_basic;
```

### 库不存在时不报错

```sql
DROP LIBRARY IF EXISTS bison_pl_c_noexists;
```

### 删除指定 schema 下的库

```sql
DROP LIBRARY sys.lib_schema;
```

### 删除替换过的库

```sql
DROP LIBRARY lib_replace;
```

### 删除库的完整流程

```sql
-- 注册共享库
CREATE LIBRARY lib_basic AS '/usr/lib/lib1.so';

-- 删除库
DROP LIBRARY lib_basic;
```
