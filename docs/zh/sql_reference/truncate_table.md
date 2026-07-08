# TRUNCATE TABLE

## 功能描述

清空表数据/释放存储

## 注意事项

- 清除其他用户的表需要TRUNCATE ANY TABLE权限
- TRUNCATE语句不能回滚
- 数据库重启回滚期间不支持

## 语法格式

**stmt:**

```sql
TRUNCATE TABLE [schema_name.]table_name [PURGE] [{DROP|REUSE} STORAGE]
```

## 参数说明

- **不指定[PURGE] [{DROP|REUSE} STORAGE]**: 清空表放入回收站，可以闪回恢复数据
- **PURGE**: 清空表并回收空间
- **DROP STORAGE**: 清空表并回收空间
- **REUSE STORAGE**: 清空表不回收空间，这部分空间自己使用

- **基本 TRUNCATE**: PURGE 回收站时回收空间，数据可闪回恢复，适用于测试开发环境。
- **TRUNCATE PURGE**: 立即回收空间，数据不可恢复，适用于空间紧张、不需要恢复数据的场景。
- **TRUNCATE DROP STORAGE**: 立即回收空间，数据不可恢复，适用于空间紧张、不需要恢复数据的场景。
- **TRUNCATE REUSE STORAGE**: 不回收空间，数据不可恢复，适用于频繁清空重载的表。

## 示例

```
-- 清空表
TRUNCATE TABLE test_data;

-- 使用 PURGE 彻底清空（不进回收站）
TRUNCATE TABLE sensitive_data PURGE;

-- 使用DROP STORAGE清空并释放空间
TRUNCATE TABLE large_table DROP STORAGE;

-- 使用REUSE STORAGE快速清空（保留分配的空间）
TRUNCATE TABLE cache_table REUSE STORAGE;

```