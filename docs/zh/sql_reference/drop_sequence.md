# DROP SEQUENCE

## 功能描述

DROP SEQUENCE用于删除数据库中的序列生成器。

## 注意事项

- 删除序列前请确认无依赖对象（如表默认值、触发器等）仍在引用该序列。
- 删除操作不可恢复，需谨慎执行。

## 语法格式

```sql
DROP SEQUENCE [IF EXISTS] [Schema.]sequence_name;
```

## 参数说明

- **IF EXISTS**: 可选，当序列不存在时不报错。
- **Schema.**: 可选，指定序列所属的模式名。
- **sequence_name**: 要删除的序列名称。

## 示例

```
-- 删除序列 MY_SEQUENCE
SQL> DROP SEQUENCE my_sequence;

Succeed.

SQL> DROP SEQUENCE IF EXISTS my_sequence;

Succeed.
```