# CREATE VIEW

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:46.853Z pushedAt=2026-08-17T06:24:20.966Z -->

## Function Description

Creates a view in a database, which is based on the SELECT query result of one or more tables. The view itself does not store data; instead, results are dynamically generated at query time.

## Precautions

Creating a view requires the query permission on the base tables. Using `OR REPLACE` overwrites the existing view definition, and a view created with `FORCE` may not be queryable when the base tables do not exist.

## Syntax

```sql
CREATE [OR REPLACE] [FORCE] VIEW view_name [(column_name [, column_name] ...)]
AS subquery;
```

## Parameter Description

| Name | Description |
|------|---------------------|
| OR REPLACE | Replaces the view if it already exists. |
| FORCE | Creates the view even if the base tables do not exist or the query is invalid. |
| view_name | Name of the view to be created. |
| column_name | List of column names for the view. |
| subquery | SELECT query statement that defines the view. |

## System Views

System-predefined views provide database runtime information and metadata. View names typically start with `DV_`.

### Common System Views

- `DV_BUFFER_POOLS`: buffer pool information

- `DV_DATA_FILES`: data file information

- `DV_TABLESPACES`: tablespace information

- `DV_PARAMETERS`: parameter information

- `DV_GLOBAL_TRANSACTIONS`: global transaction information

### Querying System Views

```
-- Query buffer pool information.
SELECT * FROM DV_BUFFER_POOLS;

-- Query system parameters.
SELECT * FROM DV_PARAMETERS;
```

## Examples

### Creating a Basic View

```
-- Create a view based on the employees table, containing only specific columns.
CREATE VIEW employee_view AS
SELECT employee_id, first_name, last_name, department_id, hire_date
FROM employees;
```

### Specifying Column Names

```
-- Create a view and specify custom column names.
CREATE VIEW employee_department_view (emp_id, emp_name, dept_id)
AS
SELECT employee_id, CONCAT(first_name, ' ', last_name), department_id
FROM employees;
```

### Creating a Conditional View

```
-- Create a view that only includes employees from the sales department.
CREATE VIEW sales_employee_view AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE department_id = 100;
```

### Creating a View Based on a Multi-Table Join

```
-- Create a view that joins the employees and departments tables.
CREATE VIEW employee_details_view AS
SELECT e.employee_id, e.first_name, e.last_name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

### Using the OR REPLACE Option

```
-- Modify the existing view.
CREATE OR REPLACE VIEW employee_view AS
SELECT employee_id, first_name, last_name, department_id, hire_date, salary
FROM employees;
```

### Using the FORCE Option

```
-- Create a view even if the base tables do not exist.
CREATE OR REPLACE FORCE VIEW future_employee_view (emp_id, emp_name, dept_id)
AS
SELECT employee_id, first_name, department_id
FROM future_employees;
```

### Creating a View with Aggregate Functions

```
-- Create a view that counts the number of employees and average salary by department.
CREATE VIEW department_statistics_view AS
SELECT department_id, COUNT(*) AS employee_count, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id;
```

### Creating a View Using the WITH AS Clause

```
-- Create a view using the WITH AS clause.
CREATE VIEW top_employees_view AS
WITH high_salary_employees AS (
    SELECT employee_id, first_name, last_name, salary
    FROM employees
    WHERE salary > 10000
)
SELECT * FROM high_salary_employees
ORDER BY salary DESC;
```
