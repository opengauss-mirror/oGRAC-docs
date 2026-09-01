# INSERT

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:37:28.674Z pushedAt=2026-07-29T06:14:15.868Z -->

## Description

`INSERT` is used to insert new data rows into a database table.

## Notes

When inserting data, the number and order of values must correspond one-to-one with the target columns; string and date values must be enclosed in single quotation marks, while numeric values do not require quotation marks. Before performing a batch insert, ensure that the number of columns in the source query matches the number of target columns.

## Syntax Format

The most basic form of the `INSERT` statement is as follows:

```sql
INSERT INTO [schema.]table_name [(column1, column2, ..., columnN)] 
VALUES (value1, value2, ..., valueN)[, (value1, value2, ..., valueN)];
```

## Parameter Description

| Name | Description |
| ------ | ------ |
| INSERT INTO ... VALUES | Keywords indicating that an insert operation is to be performed. |
| table_name | The name of the target table into which data is to be inserted. |
| [schema.] | (Optional) Specifies the schema to which the table belongs. If omitted, the schema of the current user is used by default. |
| (column1, column2, ..., columnN) | (Optional) Specifies the list of columns into which values are to be inserted. Explicitly specifying columns in the SQL statement improves readability and robustness. Even if the table structure changes, this `INSERT` statement remains valid as long as the specified columns are not affected. The order of columns does not need to match the physical order in the table definition, but the provided `VALUES` must correspond one-to-one with the column order specified here. |
| (value1, value2, ..., valueN) | Provides the specific values to be inserted into the specified columns. The number and order of values must match the columns specified in the `column_list`. |
| [, (value1, value2, ..., valueN)] | (Optional) Allows multiple rows of data to be inserted simultaneously. Each value list corresponds to one row of data, separated by commas. |

The values provided in the `VALUES` clause must follow these format requirements:

- String and date values must be enclosed in single quotation marks (' '). Numeric values do not require quotation marks.

- The `NULL` keyword can be used to explicitly insert a null value.

- Values computed using various database-compatible functions or expressions can be used.

The following uses the table `employees (id NUMBER, name VARCHAR2(50), salary NUMBER, hire_date DATE)` for illustration.

Example 1: Inserting a complete row (specifying all columns)

```sql
INSERT INTO employees (id, name, salary, hire_date) VALUES (101, 'John Doe', 50000, TO_DATE('2023-01-15', 'YYYY-MM-DD'));
```

Example 2: Inserting a partial row (specifying some columns)

```sql
INSERT INTO employees (id, name) VALUES (102, 'Jane Smith');
```

Example 3: Without specifying column names (in the order defined by the table)

```sql
-- Values must be provided for every column in the table, strictly following the column order defined during table creation.
INSERT INTO employees VALUES (103, 'Bob Johnson', 55000, SYSDATE);
```

Example 4: Insert multiple rows simultaneously without specifying column names (in table definition order)

```sql
-- Values must be provided for every column in the table, strictly following the column order defined during table creation.
INSERT INTO employees VALUES (103, 'Bob Johnson', 55000, SYSDATE),
                             (104, 'Alice Brown', 60000, SYSDATE),
                             (105, 'Charlie Davis', 70000, SYSDATE);
```

## Batch Insert

When you need to insert the results of a query into another table, the `INSERT ... SELECT` statement can be used. It is suitable for batch inserts or data migration.

```sql
INSERT INTO [schema.]target_table [(column1, column2, ..., columnN)] SELECT {subquery | (subquery)};
```

### Batch Insert Parameters

| Name | Description |
| ------ | ------ |
| [schema.]target_table | The target table. |
| (column1, column2, ..., columnN) | (Optional) The columns in the target table that receive data. |
| SELECT {subquery \| (subquery)} | The `SELECT` query that provides the source data. |

> **NOTE:** The number of columns returned by the subquery must match the number of target columns (or the total number of columns in the target table if no columns are specified). The data types must be compatible.

Example: Copying data from one table to another (with optional calculations)

```sql
-- -- Assume that the table temp_employees has the same structure as employees.
INSERT INTO employees (id, name, salary, hire_date)
SELECT id, name, salary, hire_date FROM temp_employees
WHERE processed = 'Y';

INSERT INTO bonus_log (emp_id, bonus_amount, bonus_date)
SELECT id, salary * 0.1, SYSDATE
FROM employees
WHERE department_id = 10;
```

## Examples

```sql
-- -- Insert a single complete record.
INSERT INTO employees (id, name, salary, hire_date)
VALUES (1, 'Alice', 80000, TO_DATE('2024-01-10', 'YYYY-MM-DD'));

-- -- Insert multiple records.
INSERT INTO employees (id, name, salary)
VALUES (2, 'Bob', 70000),
       (3, 'Carol', 75000);

-- -- Batch insert from query results.
INSERT INTO employees_archive (id, name, salary)
SELECT id, name, salary FROM employees WHERE hire_date < TO_DATE('2024-01-01', 'YYYY-MM-DD');
```
