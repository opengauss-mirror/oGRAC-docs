# UPDATE

## 功能描述

UPDATE 语句用于修改数据库表中已存在的行数据。

## 注意事项

省略 WHERE 子句会更新表中所有行，请谨慎操作；建议先通过 SELECT 语句确认待更新范围，并在事务中执行以便必要时回滚。

## 语法格式

```sql
UPDATE [schema.]table_name SET column1 = value1, column2 = value2, ... [WHERE condition];
```

## 参数说明

| 参数 | 说明 |
|------|------|
| UPDATE | 关键字，指示要执行更新操作。 |
| [schema.]table_name | 要更新数据的目标表的名称。 |
| SET column1 = value1, column2 = value2, ... | 指定要更新的列及其新值。指定更新的新值可以是常量（包括`DEFAULT`）、表达式、函数调用等。 |
| [WHERE condition] | （可选）指定更新哪些行的条件。如果省略，将更新表中的所有行。 |

示例1: 更新符合条件的行

```
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 10;
UPDATE employees SET salary = DEFAULT WHERE department_id = 10;
```

示例2: 更新所有行 (谨慎使用)

```
UPDATE employees SET name = 'Marketing';
```

## 示例

```sql
-- 根据条件更新单列
UPDATE employees SET salary = 90000 WHERE id = 101;

-- 同时更新多列
UPDATE employees SET salary = salary * 1.05, hire_date = SYSDATE WHERE department_id = 20;

-- 使用子查询更新
UPDATE employees SET salary = (SELECT AVG(salary) FROM employees) WHERE department_id = 30;
```
