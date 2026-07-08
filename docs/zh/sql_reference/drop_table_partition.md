# DROP TABLE PARTITION

## 功能描述

删除分区表中的指定分区及其数据。

## 注意事项

- 执行该语句需要拥有对应表的 ALTER TABLE 权限
- 删除分区会同时删除该分区中存储的所有数据，且不可通过回收站恢复分区本身
- 分区表中必须保留至少一个分区，不能删除唯一的分区
- 如果其他对象（如索引、视图）依赖该分区，删除前需要先处理相关依赖
- oGRAC 重启回滚期间不支持删除分区

## 语法格式

**stmt:**

```sql
ALTER TABLE [schema_name.]table_name
    DROP PARTITION partition_name
```

## 参数说明

- **schema_name**: 分区表所属的模式名，省略时默认为当前用户
- **table_name**: 要删除分区的分区表名
- **partition_name**: 要删除的分区名

## 示例

```
-- 删除范围分区表中的指定分区
ALTER TABLE sales_range DROP PARTITION sales_q1;

-- 删除列表分区表中的指定分区
ALTER TABLE employees_list DROP PARTITION dept_hr;

-- 删除指定模式下分区表的分区
ALTER TABLE hr.sales_range DROP PARTITION sales_2023_q4;
```