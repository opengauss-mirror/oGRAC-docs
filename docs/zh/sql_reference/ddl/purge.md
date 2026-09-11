# PURGE

## 功能描述

从回收站中清除表、索引、分区等对象。PURGE 支持清除表、索引、表空间和回收站，清除后的对象不再保留在回收站中。

## 注意事项

- 默认可清除自身 schema 下的对象，清除其他用户下的对象需要对应的权限：清除表需要 DROP ANY TABLE，清除索引需要 DROP ANY INDEX，清除表空间需要 DROP TABLESPACE，清空其他用户的回收站需要 PURGE DBA_RECYCLEBIN
- 带字符串 'object_name' 的形式按回收站对象名清除，回收站对象名可通过视图 SYS_RECYCLEBIN 查询
- PURGE TABLE ... PARTITION ... 用于清除指定分区

## 语法格式

```sql
PURGE TABLE [schema_name.]table_name [ PARTITION partition_name ];
PURGE TABLE 'object_name';
PURGE INDEX [schema_name.]index_name;
PURGE INDEX 'object_name';
PURGE PARTITION 'object_name';
PURGE TABLESPACE tablespace_name;
PURGE RECYCLEBIN;
```

## 参数说明

- **PURGE TABLE [schema_name.]table_name [PARTITION partition_name]**: 清除指定表；带 PARTITION 时清除指定分区
- **PURGE TABLE 'object_name'**: 按回收站对象名清除表
- **PURGE INDEX [schema_name.]index_name**: 清除指定索引
- **PURGE INDEX 'object_name'**: 按回收站对象名清除索引
- **PURGE PARTITION 'object_name'**: 按回收站对象名清除分区
- **PURGE TABLESPACE tablespace_name**: 清除指定表空间中的回收站对象
- **PURGE RECYCLEBIN**: 清空当前用户的回收站

## 示例

### 清除自身 schema 下的表

```sql
CREATE TABLE bison_ddl_user.purge_local_tab(a int);
ALTER SESSION SET current_schema = bison_ddl_user;
DROP TABLE purge_local_tab;
PURGE TABLE purge_local_tab;
```

### 清除其他用户下的表

```sql
CREATE TABLE bison_ddl_user.purge_owner_tab(a int);
DROP TABLE bison_ddl_user.purge_owner_tab;
PURGE TABLE bison_ddl_user.purge_owner_tab;
```

### 清除索引

```sql
PURGE INDEX idx_emp_lastname;
```

### 按回收站对象名清除

```sql
PURGE TABLE 'purge_owner_tab';
PURGE INDEX 'idx_emp_lastname';
PURGE PARTITION 'p1_2023';
```

### 清除表空间中的回收站对象

```sql
PURGE TABLESPACE ts1;
```

### 清空回收站

```sql
PURGE RECYCLEBIN;
```

### 查看回收站

```sql
SELECT * FROM SYS_RECYCLEBIN;
```
