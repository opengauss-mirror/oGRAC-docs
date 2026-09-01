# UPDATE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:37:23.398Z pushedAt=2026-07-29T06:14:15.860Z -->

## Description

The `UPDATE` statement is used to modify existing row data in a database table.

## NOTE

Omitting the `WHERE` clause will update all rows in the table. Exercise caution. It is advised to first verify the scope of the update using a `SELECT` statement and to perform the operation within a transaction so that it can be rolled back when necessary.

## Syntax

```sql
UPDATE [schema.]table_name SET column1 = value1, column2 = value2, ... [WHERE condition];
```

## Parameter Description

| Name | Description |
| ------ | ------ |
| UPDATE | Keyword indicating that an update operation is to be performed. |
| [schema.]table_name | Name of the target table whose data is to be updated. |
| SET column1 = value1, column2 = value2, ... | Specifies the columns to be updated and their new values. The new values can be constants (including `DEFAULT`), expressions, function calls, etc. |
| [WHERE condition] | (Optional) Specifies the condition for which rows to update. If omitted, all rows in the table will be updated. |

Example 1: Updating rows that meet conditions

```
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 10;
UPDATE employees SET salary = DEFAULT WHERE department_id = 10;
```

Example 2: Updating all rows (use with caution)

```
UPDATE employees SET name = 'Marketing';
```

## Examples

```sql
-- Update a single column based on a condition
UPDATE employees SET salary = 90000 WHERE id = 101;

-- Update multiple columns simultaneously
UPDATE employees SET salary = salary * 1.05, hire_date = SYSDATE WHERE department_id = 20;

-- Update using a subquery
UPDATE employees SET salary = (SELECT AVG(salary) FROM employees) WHERE department_id = 30;
```
