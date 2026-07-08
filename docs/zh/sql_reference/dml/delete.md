# DELETE

## 功能描述

DELETE 语句用于从数据库表中删除符合指定条件的行。

## 注意事项

省略 WHERE 子句会删除表中所有行，请谨慎操作；建议在执行前先用 SELECT 确认待删除数据，并在事务中运行以便必要时回滚。

## 语法格式

```sql
DELETE FROM [schema.]table_name [WHERE condition];
```

## 参数说明

| 参数 | 说明 |
|------|------|
| DELETE FROM ... | 关键字，指示要执行删除操作。 |
| [schema.]table_name | 要删除数据的目标表的名称。 |
| [WHERE condition] | （可选）指定删除哪些行的条件。如果省略，将删除表中的所有行。 |

示例1: 删除所有符合条件的行

```
DELETE FROM employees WHERE id = 10;
DELETE FROM employees WHERE id IN (SELECT id FROM temp_employees WHERE processed = 'Y');
```

示例2: 删除所有行 (谨慎使用)

```
DELETE FROM employees;
```

## 示例

```sql
-- 根据主键删除单行
DELETE FROM employees WHERE id = 100;

-- 根据范围条件删除
DELETE FROM employees WHERE salary < 30000;

-- 使用子查询删除
DELETE FROM employees WHERE department_id IN (SELECT department_id FROM departments WHERE location = 'Old Office');
```
