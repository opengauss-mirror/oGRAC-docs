# WHERE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:35:53.002Z pushedAt=2026-07-29T06:14:15.828Z -->

## Description

`WHERE` is primarily used to filter data rows from a table that meet specified conditions. Only rows that satisfy the conditions of the `WHERE` clause are returned.

## Precautions

Avoid using functions or implicit conversions on index columns in the `WHERE` clause, as this may prevent index utilization. When combining multiple conditions, it is recommended to use parentheses to explicitly specify operation precedence.

## Syntax

```sql
SELECT 
    { * | column1 [, column2, ...] }
[ FROM table_name [, ...] ]
[ WHERE condition ];
```

## Parameter Description

- **WHERE clause**: A row selection expression used to restrict the query scope of a `SELECT` statement. Comparison operators, logical operators, and special conditional expressions can be used in the `WHERE` clause.

- **condition**: Any expression that returns a Boolean value. Only rows that satisfy this condition (with a result of `TRUE`) are retrieved.

## Detailed Breakdown of condition Components

- **Comparison expression**: `column comparison_operator value`

    Comparison operators include:

  - **=**

  - **>**

  - **<**

  - **>=**

  - **<=**

  - **<>** or **!=**

    Examples:

    ```sql
    SELECT * FROM customer_t1 WHERE c_customer_sk > 1000;
    SELECT * FROM customer_t1 WHERE c_customer_id = 'hello';
    SELECT * FROM customer_t1 WHERE c_customer_sk <> 3869;
    ```

- **Logical expression**: `condition1 logical_operator condition2`

    Logical operators include:

  - **AND**: Both conditions are true simultaneously.

  - **OR**: At least one of the two conditions is true.

  - **NOT**: Negates the condition.

    Example:

    ```sql
    SELECT * FROM customer_t1 WHERE c_customer_sk > 1000 AND c_customer_id = 'hello';
    SELECT * FROM customer_t1 WHERE c_customer_sk > 6985 OR c_customer_id = 'hello';
    SELECT * FROM customer_t1 WHERE NOT c_customer_id = 'hello';
    ```

- **BETWEEN AND range expression**: `column BETWEEN value1 AND value2`

    Equivalent to `column >= value1 AND column <= value2`

    Example:

    ```sql
    SELECT * FROM customer_t1 WHERE c_customer_sk BETWEEN 4000 AND 9000;
    SELECT * FROM customer_t1 WHERE c_customer_sk NOT BETWEEN 4000 AND 9000;
    ```

- **IN set expression**: `column IN (value1, value2, ...)`

    Determines whether a column value is in the specified set.

    Example:

    ```sql
    SELECT * FROM customer_t1 WHERE c_customer_sk IN (1000, 2000, 3000);
    SELECT * FROM customer_t1 WHERE c_customer_id IN ('hello', 'world', 'test');
    SELECT * FROM customer_t1 WHERE c_customer_sk NOT IN (1000, 2000, 3000);
    ```

- **LIKE pattern matching expression**: `column LIKE pattern [ESCAPE 'escape_char']`

    Pattern matching wildcards:

  - **%**: Matches any number of characters (including zero).

  - **_**: Matches a single character

    `ESCAPE` character:

    When you need to match the wildcard characters themselves (% or _), you can use the `ESCAPE` clause to specify an escape character, which converts the wildcard into a literal character.

    Example:

    ```sql
    SELECT * FROM customer_t1 WHERE c_customer_id LIKE 'A%';
    SELECT * FROM customer_t1 WHERE c_customer_id LIKE 'A_';
    SELECT * FROM customer_t1 WHERE c_customer_id LIKE 'A\%' ESCAPE '\';
    ```

- **NULL check expression**: `column IS NULL`

    `column IS NOT NULL`

    Determines whether a column value is NULL.

    Example:

    ```sql
    SELECT * FROM customer_t1 WHERE c_last_name IS NULL;
    SELECT * FROM customer_t1 WHERE c_last_name IS NOT NULL;
    ```

## Index Scan

Index scans are supported for basic comparison operators and statements using `LIKE`, `IN`, `BETWEEN AND`, `IS [NOT] NULL`, `TRUE`, and `FALSE`.

When functions, implicit conversions, or leading wildcard `LIKE` are applied to indexed columns, the cost of index scans cannot be calculated.

In the absence of statistical information, the cost of index scans cannot be calculated.

For example:

```sql
SELECT * FROM customer_t1 WHERE c_customer_sk + 1 = 1000;       -- No index scan
SELECT * FROM customer_t1 WHERE c_customer_id LIKE '%A';        -- Leading wildcard LIKE does not use index scan
SELECT * FROM customer_t1 WHERE c_customer_sk LIKE '1%';    -- Implicit conversion does not use index scan
```

## Examples

```sql
-- Use comparison operators
SELECT * FROM employees WHERE salary > 50000;

-- Use logical operators
SELECT * FROM employees WHERE department_id = 10 AND salary > 60000;

-- Use BETWEEN and IN
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;
SELECT * FROM employees WHERE department_id IN (10, 20);

-- Use LIKE matching
SELECT * FROM employees WHERE name LIKE 'M%';

-- Check NULL
SELECT * FROM employees WHERE manager_id IS NULL;
```
