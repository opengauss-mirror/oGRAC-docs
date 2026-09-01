# DELETE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:37:06.285Z pushedAt=2026-07-29T06:14:15.855Z -->

## Function Description

The `DELETE` statement is used to delete rows that meet specified conditions from a database table.

## Precautions

Omitting the `WHERE` clause will delete all rows in the table. Proceed with caution. It is advised to use `SELECT` to verify the data to be deleted before execution, and to run the operation within a transaction so that it can be rolled back if necessary.

## Syntax

```sql
DELETE FROM [schema.]table_name [WHERE condition];
```

## Parameter Description

| Name | Description |
| ------ | ------ |
| DELETE FROM ... | Keyword indicating that a delete operation is to be performed. |
| [schema.]table_name | Name of the target table from which data is to be deleted. |
| [WHERE condition] | (Optional) Specifies the condition for which rows to delete. If omitted, all rows in the table will be deleted. |

Example 1: Deleting all rows that meet the condition

```
DELETE FROM employees WHERE id = 10;
DELETE FROM employees WHERE id IN (SELECT id FROM temp_employees WHERE processed = 'Y');
```

Example 2: Deleting all rows (proceed with caution)

```
DELETE FROM employees;
```

## Examples

```sql
-- Delete a single row by primary key.
DELETE FROM employees WHERE id = 100;

-- Delete rows based on a range condition.
DELETE FROM employees WHERE salary < 30000;

-- Delete rows using a subquery.
DELETE FROM employees WHERE department_id IN (SELECT department_id FROM departments WHERE location = 'Old Office');
```
