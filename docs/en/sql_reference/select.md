# SELECT

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:17:18.750Z pushedAt=2026-08-18T10:51:00.101Z -->

## Function Description

The SELECT operation is read-only and does not modify any data in the database. However, it can combine and transform data in various ways, providing the foundation for report generation, data analysis, and application queries.

## Precautions

For complex SELECT statements with many clauses and joins, ensure proper matching between column aliases, grouping columns, and ordering columns. Also note that `FOR UPDATE` locks result rows and can reduce concurrency.

## Syntax

```sql
SELECT [DISTINCT] column1, column2, ... | [*] | expression
FROM TABLE_NAME | VIEW_NAME | SUBQUERY
[WHERE condition]
[ [ START WITH condition ] CONNECT BY [NOCYCLE] condition [ ORDER SIBLINGS BY expression ] ]
[GROUP BY {cube | rollup} column_list] 
[HAVING condition]
[ { UNION | INTERSECT | EXCEPT | MINUS } [ ALL | DISTINCT ] select ]
[ORDER BY column_list | expression [ASC|DESC]] 
[LIMIT offset, count]
[ {FOR { UPDATE | NO KEY UPDATE | SHARE | KEY SHARE } [ OF table_name [, ...] ] [ NOWAIT | WAIT N]} [...] ];
```

The following examples are based on the table:

```
CREATE TABLE employees (
    employee_id NUMBER PRIMARY KEY,
    name VARCHAR2(20) NOT NULL,
    manager_id NUMBER,
    department_id NUMBER,
    job_title VARCHAR2(20),
    salary NUMBER,
    hire_date DATE
);
INSERT INTO employees VALUES (1, 'CEO', NULL, 10, 'CEO', 150000, SYSDATE);
INSERT INTO employees VALUES (2, 'Manager1', 1, 10, 'Manager', 100000, SYSDATE);
INSERT INTO employees VALUES (3, 'Manager2', 1, 20, 'Manager', 95000, SYSDATE);
INSERT INTO employees VALUES (4, 'Developer1', 2, 10, 'Developer', 85000, SYSDATE);
INSERT INTO employees VALUES (5, 'Developer2', 2, 10, 'Developer', 80000, SYSDATE);
COMMIT;
```

## Parameter Description

| Name | Description |
|------|------|
| DISTINCT | Optional. Removes duplicate rows. |
| column1, column2, ... / * / expression | Specifies the columns or expressions to query. |
| FROM | Specifies the data source. |
| WHERE | Specifies row filtering conditions. |
| GROUP BY | Specifies grouping. |
| HAVING | Specifies group filtering conditions. |
| ORDER BY | Specifies result sorting. |
| LIMIT | Limits the number of returned rows. |
| FOR UPDATE / SHARE ... | Locking clause. |

- **[DISTINCT]**: (Optional) Removes duplicate rows from the query result.

    - When DISTINCT is used together with ORDER BY, the columns specified in ORDER BY must be included in the columns of the result set retrieved by the SELECT statement.

    ```
    SQL> SELECT DISTINCT department_id, job_title FROM employees ORDER BY department_id, job_title;

    DEPARTMENT_ID                            JOB_TITLE           
    ---------------------------------------- --------------------
    10                                       CEO                 
    10                                       Developer           
    10                                       Manager             
    20                                       Manager             

    4 rows fetched.
    ```

- **column1, column2, ... | [*] | expression**: Specifies the result to query.

    - [*] The asterisk \* can be used to query all columns.

    - `expression` can be a constant, expression, function call, etc.

    ```
    SELECT * FROM employees;
    SELECT department_id, job_title FROM employees;

    SQL> SELECT department_id + 100, UPPER(job_title) FROM employees WHERE department_id = 10;

    DEPARTMENT_ID + 100                      UPPER(JOB_TITLE)    
    ---------------------------------------- --------------------
    110                                      CEO                 
    110                                      MANAGER             
    110                                      DEVELOPER           
    110                                      DEVELOPER           

    4 rows fetched.

    ```

- **FROM TABLE_NAME | VIEW_NAME | SUBQUERY**: Specifies the data source to query, which can be a table, view, or subquery.

    ```
    SQL> CREATE OR REPLACE VIEW employee_view AS SELECT * FROM employees WHERE department_id = 10;

    Succeed.

    SQL> SELECT department_id, name FROM employee_view WHERE job_title = 'Manager';

    DEPARTMENT_ID                            NAME                
    ---------------------------------------- --------------------
    10                                       Manager1            

    1 rows fetched.

    SQL> SELECT department_id, name FROM (SELECT * FROM employees WHERE department_id = 10) WHERE job_title = 'Manager';

    DEPARTMENT_ID                            NAME                
    ---------------------------------------- --------------------
    10                                       Manager1            

    1 rows fetched.
    ```

- **[WHERE condition]**: (Optional) The WHERE clause forms a row selection expression that narrows the scope of the SELECT query. `condition` is any expression that returns a Boolean value. Any row that does not satisfy this condition will not be retrieved.

    - **Common conditions include**: =, <>[!=], >, <, >=, <=, AND, OR, BETWEEN AND, IN, LIKE [ESCAPE], IS [NOT] NULL, [NOT] IN, [NOT] EXISTS, etc.

    ```sql
    SELECT * FROM employees WHERE department_id = [!=] | [>] | [<] | [>=] | [<=] 10;
    SELECT * FROM employees WHERE job_title [NOT] LIKE '%Sales\%' ESCAPE '\';
    SELECT * FROM employees WHERE salary [NOT] BETWEEN 5000 AND 10000;
    SELECT * FROM employees WHERE manager_id IS [NOT] NULL;
    SELECT * FROM employees WHERE department_id [NOT] IN (10, 20, 30);
    ```

- **[START WITH condition]**: (Optional) Specifies the starting condition for a recursive query.

    - The START WITH clause is typically used together with the CONNECT BY clause to perform hierarchical recursive traversal queries on data, where START WITH represents the initial condition of the recursion. If this clause is omitted and the CONNECT BY clause is used alone, all rows in the table are treated as the initial set.

- **[CONNECT BY [NOCYCLE] condition]**: (Optional) Specifies the join condition for a recursive query.

    - CONNECT BY represents the recursive join condition and is used together with the START WITH clause to implement data traversal and recursion. In addition, the PRIOR keyword can also be used in the target column, allowing users to obtain the value of the previous level through the target column.

- **[ORDER SIBLINGS BY expression]**: (Optional) Sorts the child rows of the recursive query result.

    - ORDER SIBLINGS BY is typically used together with the START WITH and CONNECT BY clauses. Its usage is the same as that of the ORDER BY clause, and it is used for hierarchical sorting during recursion.

    ```
    SQL> SELECT LPAD(' ', 2 * LEVEL) || name AS employee_name, department_id
    2 FROM employees
    3 START WITH manager_id IS NULL
    4 CONNECT BY PRIOR employee_id = manager_id
    5 ORDER SIBLINGS BY name;

    EMPLOYEE_NAME                                                    DEPARTMENT_ID                           
    ---------------------------------------------------------------- ----------------------------------------
    CEO                                                              10                                      
        Manager1                                                     10                                      
            Developer1                                               10                                      
            Developer2                                               10                                      
        Manager2                                                     20                                      

    5 rows fetched.
    ```

- **[GROUP BY {cube | rollup} column_list]**: (Optional) Used to group query results.

    - **GROUPING SETS ( grouping_element [, ...] )**: The GROUPING SETS clause is a further extension of the GROUP BY clause. It allows users to specify multiple GROUP BY options. This improves efficiency by pruning data groups that users do not need.

    ```
    SQL> SELECT department_id, job_title, SUM(salary) AS total_salary
    2 FROM employees
    3 GROUP BY department_id, job_title
    4 ORDER BY department_id, job_title;

    DEPARTMENT_ID                            JOB_TITLE            TOTAL_SALARY                            
    ---------------------------------------- -------------------- ----------------------------------------
    10                                       CEO                  150000                                  
    10                                       Developer            165000                                  
    10                                       Manager              100000                                  
    20                                       Manager              95000                                   

    4 rows fetched.
    ```

    - **CUBE (expression, ...)**: CUBE automatically performs grouped aggregation on the columns listed in the GROUP BY clause. The result set contains all possible combinations of values from the dimension columns, along with aggregated values from the base rows that match each combination. It returns one summary row per group, and can be used to generate cross-tabulation values. For example, if three expressions (n = 3) are specified in the CUBE clause, the operation produces 2^n = 2^3 = 8 groups. Rows grouped by all n expressions are called regular rows, and the remaining rows are called super-aggregate rows.

    ```
    -- GROUP BY CUBE logical equivalence
    -- CUBE (c1,c2,c3) <--> GROUPING SETS ((c1, c2, c3),(c1, c2),(c2, c3),(c1, c3),(c1),(c2),(c3),())
    SQL> SELECT department_id, job_title, SUM(salary) AS total_salary
    2 FROM employees
    3 GROUP BY CUBE(department_id, job_title)
    4 ORDER BY department_id, job_title;

    DEPARTMENT_ID                            JOB_TITLE            TOTAL_SALARY                            
    ---------------------------------------- -------------------- ----------------------------------------
    10                                       CEO                  150000                                  
    10                                       Developer            165000                                  
    10                                       Manager              100000                                  
    10                                                            415000                                  
    20                                       Manager              95000                                   
    20                                                            95000                                   
                                             CEO                  150000                                  
                                             Developer            165000                                  
                                             Manager              195000                                  
                                                                  510000                                  

    10 rows fetched.
    ```

    - ROLLUP is a shortcut for generating multiple grouping sets. Unlike the CUBE clause, ROLLUP does not generate all possible grouping sets based on specific columns; instead, it generates a subset of them. ROLLUP assumes a hierarchical structure among the input columns and generates only the meaningful grouping sets accordingly.

    ```
    -- GROUP BY ROLLUP logical equivalence
    -- ROLLUP(c1,c2,c3) <--> GROUPING SETS ((c1, c2, c3),(c1, c2),(c1),())
    SQL> SELECT department_id, job_title, SUM(salary) AS total_salary
    2 FROM employees
    3 GROUP BY ROLLUP(department_id, job_title)
    4 ORDER BY department_id, job_title;

    DEPARTMENT_ID                            JOB_TITLE            TOTAL_SALARY                            
    ---------------------------------------- -------------------- ----------------------------------------
    10                                       CEO                  150000                                  
    10                                       Developer            165000                                  
    10                                       Manager              100000                                  
    10                                                            415000                                  
    20                                       Manager              95000                                   
    20                                                            95000                                   
                                                                  510000                                  

    7 rows fetched.
    ```

    - If an expression in the SELECT list references columns that are not included in the GROUP BY clause, the SQL statement will report an error, unless an aggregate function is applied to those columns or the columns are functionally dependent on the grouping columns. This is because ungrouped columns may return multiple values.

- **[HAVING condition]**: (Optional) Filters the grouped results.

    - HAVING works with GROUP BY to filter groups. It compares group attributes with constant values, and only those groups that satisfy the logical expression in the HAVING clause are retrieved.

    ```
    SQL> SELECT department_id, AVG(salary) AS avg_salary
    2 FROM employees
    3 GROUP BY department_id
    4 HAVING AVG(salary) > 85000 -- Filter after grouping.
    5 ORDER BY department_id;

    DEPARTMENT_ID                            AVG_SALARY                              
    ---------------------------------------- ----------------------------------------
    10                                       103750                                  
    20                                       95000                                   

    2 rows fetched.
    ```

- **[{ UNION | INTERSECT | EXCEPT | MINUS } [ ALL | DISTINCT ] SELECT]**: (Optional) Performs set operations on the result sets of multiple queries. The number of columns and their data types on both sides of the subqueries must be consistent or compatible.

    - UNION computes the union of the row sets returned by multiple SELECT statements.

        - Unless the ALL clause is specified, the default UNION result does not include duplicate rows.

        - Multiple UNION operators within the same SELECT statement are evaluated from left to right, unless precedence is identified by parentheses.

    ```sql
    SELECT employee_id, name, department_id FROM employees WHERE department_id = 10
    UNION [ALL]
    SELECT employee_id, name, department_id FROM employees WHERE department_id = 20 AND job_title = 'Manager';
    ```

    - INTERSECT computes the intersection of the row sets returned by multiple SELECT statements.

        - Unless the ALL clause is declared, the default INTERSECT result does not contain duplicate rows.

        - Multiple INTERSECT operators within the same SELECT statement are evaluated from left to right, unless precedence is identified by parentheses.

        - When UNION and INTERSECT operations are performed on the execution results of multiple SELECT statements, INTERSECT is processed first.

    ```sql
    SELECT employee_id, name, department_id FROM employees WHERE department_id = 10
    INTERSECT [ALL]
    SELECT employee_id, name, department_id FROM employees WHERE department_id = 20 AND job_title = 'Manager';
    ```

    - EXCEPT computes the difference set of the row sets returned by multiple SELECT statements.

        - Unless the ALL clause is declared, the default EXCEPT result does not contain duplicate rows.

        - The EXCEPT operator computes rows that exist in the output of the SELECT statement on the left but do not exist in the output of the SELECT statement on the right.

        - When ALL is used, a row that has m duplicates in the left table and n duplicates in the right table will appear max(m-n,0) times in the result.

        - Multiple EXCEPT operators in the same SELECT statement are evaluated from left to right, unless precedence is identified by parentheses.

        - EXCEPT and UNION have the same binding level.

    ```sql
    SELECT employee_id, name, department_id FROM employees WHERE department_id = 10
    EXCEPT [ALL]
    SELECT employee_id, name, department_id FROM employees WHERE department_id = 20 AND job_title = 'Manager';
    ```

    - MINUS computes the difference set of row sets returned by multiple SELECT statements. It functions the same as the EXCEPT operator, but cannot be used together with the ALL clause.

- **[ORDER BY column_list | expression [ASC|DESC]]**: (Optional) Sorts the query results.

    - If the ORDER BY expression contains multiple columns or expressions, the sorting precedence is from left to right.

    - When ORDER BY is used together with the DISTINCT keyword, the columns specified in ORDER BY must be included in the columns of the result set retrieved by the SELECT statement.

        - ORDER BY can support expressions, where the referenced columns must be present in DISTINCT.

    - When ORDER BY is used together with GROUP BY, the columns specified in ORDER BY must be included in the columns of the result set retrieved by the SELECT statement.

- **[LIMIT offset, count]**: (Optional) Limits the number of rows returned in the query result. `offset` specifies the starting position, and `count` specifies the number of rows to return.

    - When only one parameter is specified in the LIMIT clause, the return starts from the first row of the result set by default.

    ```
    SQL> SELECT employee_id, name, department_id, salary
    2 FROM employees
    3 ORDER BY employee_id
    4 LIMIT 2, 2;  -- Skip the first 2 rows and return the next 2 rows.

    EMPLOYEE_ID                              NAME                 DEPARTMENT_ID                            SALARY                                  
    ---------------------------------------- -------------------- ---------------------------------------- ----------------------------------------
    3                                        Manager2             20                                       95000                                   
    4                                        Developer1           10                                       85000                                   

    2 rows fetched.
    ```

- **[FOR UPDATE]**: (Optional) Locks selected rows during the query.

    ```sql
    SELECT employee_id, name, department_id, salary
    FROM employees
    ORDER BY employee_id
    FOR UPDATE;  -- Row-level locks on all returned rows
    ```

## JOIN Syntax

JOIN combines multiple tables. It brings together data from different tables based on join conditions to form a meaningful result set.

Basic syntax structure

```
SELECT column1, column2, ... | expression | function
FROM [Schema.]{table1 | view1 | subquery1} [alias1]
{JOIN TYPE} JOIN [Schema.]{table2 | view2 | subquery2} [alias2] 
    ON alias1.{column | expression | function} = alias2.{column | expression | function}
{JOIN TYPE} JOIN [Schema.]{table3 | view3 | subquery3} [alias3] 
    ON alias2.{column | expression | function} = alias3.{column | expression | function}
···
{JOIN TYPE} JOIN [Schema.]{tableN | viewN | subqueryN} [aliasN] 
    ON alias(N-1).{column | expression | function} = aliasN.{column | expression | function}
[WHERE conditions]
[GROUP BY columns]
[HAVING conditions]
[ORDER BY columns];
```

- **{JOIN TYPE}**: Specifies the join type, including [ INNER ] JOIN, LEFT [ OUTER ] JOIN, RIGHT [ OUTER ] JOIN, FULL [ OUTER ] JOIN, and others.

    - **INNER JOIN**: Returns rows where the join condition is fully matched in both tables, that is, the intersection of the two tables.

    - **LEFT [ OUTER ] JOIN**: Returns all rows from the left table (the first table after FROM), even if there are no matching rows in the right table. Unmatched portions in the right table are padded with NULL.

    - **RIGHT [ OUTER ] JOIN**: Returns all rows from the right table (the second table after JOIN), even if there are no matching rows in the left table. Unmatched portions in the left table are padded with NULL.

    - **FULL [ OUTER ] JOIN**: Returns all rows from both the left and right tables, with unmatched portions padded with NULL.

- **aliasN**: (Optional) Specifies an alias for a table, view, or subquery for subsequent reference in the query.

- **alias(N-1).{column | expression | function} = aliasN.{column | expression | function}**: Specifies the join condition, which defines the matching between two tables. The matching elements can be columns, expressions, functions, etc.

### [ INNER ] JOIN

- Returns rows where the join condition is fully matched between the two tables, i.e., the intersection of the two tables.

- INNER can be omitted without changing the semantics.

```
create table t_join_1(a int, b int, c int);
create table t_join_2(a int, b int, c int);
insert into t_join_1 values(1,1,1),(2,2,2),(3,3,3),(4,4,4);
insert into t_join_2 values(1,1,1),(2,2,2),(3,3,3),(5,5,5);

-- Use table t_join_1 as the left table and t_join_2 as the right table, and perform an inner join on columns a and b.
SQL> select * from t_join_1 t1 inner join t_join_2 t2 on t1.a = t2.b;

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
2            2            2            2            2            2           
3            3            3            3            3            3           

3 rows fetched.
```

### LEFT [ OUTER ] JOIN

- Returns all rows from the left table (the first table after FROM and before LEFT JOIN), even if there are no matching rows in the right table. Unmatched portions in the right table are filled with NULL.

- OUTER can be omitted without changing the semantics.

```
-- Use table t_join_1 as the left table and t_join_2 as the right table, and perform a left outer join on columns a and b.
SQL> select * from t_join_1 t1 left outer join t_join_2 t2 on t1.a = t2.b;

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
2            2            2            2            2            2           
3            3            3            3            3            3           
4            4            4                                                  

4 rows fetched.

```

### RIGHT [ OUTER ] JOIN

- Semantically opposite to LEFT [ OUTER ] JOIN. Returns all rows from the right table (the first table after JOIN), even if there are no matching rows in the left table. Unmatched portions in the left table are filled with NULL.

- OUTER can be omitted without changing the semantics.

- This is merely a notational convenience, because any RIGHT [ OUTER ] JOIN can always be converted into a LEFT OUTER JOIN by simply swapping the left and right inputs.

```
-- Use table t_join_1 as the left table and t_join_2 as the right table, and perform a right outer join on columns a and b.

SQL> select * from t_join_1 t1 right outer join t_join_2 t2 on t1.a = t2.b;

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
2            2            2            2            2            2           
3            3            3            3            3            3           
                                         5            5            5

4 rows fetched.

```

### FULL [ OUTER ] JOIN

- Returns all rows that satisfy the join condition, plus unmatched rows from the left table (extended with NULL for the right table), and unmatched rows from the right table (extended with NULL for the left table).

- OUTER can be omitted without changing the semantics.

```
-- Use table t_join_1 as the left table and t_join_2 as the right table, and perform a full outer join on columns a and b.

SQL> select * from t_join_1 t2 full outer join t_join_2 t1 on t2.b = t1.a;

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
2            2            2            2            2            2           
3            3            3            3            3            3           
4            4            4                                                  
                                         5            5            5

5 rows fetched.

```

### CROSS JOIN

- Returns the Cartesian product of two tables (all possible row combinations) without specifying a join condition, meaning there is no ON clause in the SQL statement.

- CROSS JOIN is equivalent to INNER JOIN ON (TRUE), that is, all row combinations of the left and right tables. This join type is merely a notational convenience, as it produces the same result as a simple FROM and WHERE.

- The number of rows in the final result set is the product of the number of rows in the left table and the number of rows in the right table.

```
-- Use table t_join_1 as the left table and table t_join_2 as the right table to perform a cross join.

SQL> select * from t_join_1 t1 cross join t_join_2 t2;

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
1            1            1            2            2            2           
1            1            1            3            3            3           
1            1            1            5            5            5           
2            2            2            1            1            1           
2            2            2            2            2            2           
2            2            2            3            3            3           
2            2            2            5            5            5           
3            3            3            1            1            1           
3            3            3            2            2            2           
3            3            3            3            3            3           
3            3            3            5            5            5           
4            4            4            1            1            1           
4            4            4            2            2            2           
4            4            4            3            3            3           
4            4            4            5            5            5           

16 rows fetched.

```

## WHERE (+) Syntax

oGRAC supports converting table join relationships into outer joins by specifying the "(+)" operator in the WHERE clause. This syntax is not standard SQL syntax, and syntax compatibility issues may arise during platform migration. In addition, using "(+)" has many restrictions:

- "(+)" can only appear in the WHERE clause. If a table join relationship has already been specified in the FROM clause, "(+)" cannot be used in the WHERE clause.

```
-- Error example: A join relationship has already been specified in the FROM clause, so "(+)" cannot be used in the WHERE clause, otherwise an error will be reported.

SQL> SELECT * FROM t_join_1 t1 JOIN t_join_2 t2 ON t1.a = t2.a WHERE t1.b = t2.b (+); 

OG-00601, Sql syntax error: the 'join' keyword is not allowed when exist '(+)'

```

- "(+)" can only act on columns of a table or view, and cannot act on expressions.

```
-- Error example: Using "(+)" in an expression causes an error.

SQL> SELECT * FROM t_join_1 t1, t_join_2 t2 WHERE (t1.a + 1)(+) = t2.a;

OG-00601, [1:57]Sql syntax error: invalid word '(+)' found

SQL> SELECT * FROM t_join_1 t1, t_join_2 t2 WHERE (t1.a + 1) = t2.a(+);

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            2            2            2           
2            2            2            3            3            3           
3            3            3                                                  
4            4            4            5            5            5           

4 rows fetched.

```

- If table A and table B have multiple join conditions, "(+)" must be specified in all join conditions. Otherwise, "(+)" will not take effect, the table join will be converted to an inner join, and no notification will be given.

```
-- Error example: Using "(+)" only in some conditions converts the join to an inner join without any error message.

SQL> SELECT * FROM t_join_1 t1, t_join_2 t2 WHERE t1.a = t2.a (+) AND t1.b = t2.b;

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
2            2            2            2            2            2           
3            3            3            3            3            3           

3 rows fetched.

```

- Expressions with "(+)" cannot be directly connected by "OR".

```
-- Error example: Directly connecting "(+)" expressions with "OR" causes an error.
SQL> SELECT * FROM t_join_1 t1, t_join_2 t2 WHERE t1.a = t2.a(+) OR t1.b = t2.b; 

OG-00601, [1:53]Sql syntax error: invalid usage outer join symbol
```

- If the column on which "(+)" operates is compared with a constant, the expression becomes part of the join condition.

```

SQL> SELECT * 
  2 FROM t_join_1, t_join_2 
  3 WHERE t_join_1.a = t_join_2.a (+)  -- Normal join condition
  4   AND t_join_2.b (+) = 1;          -- (t_join_2.b = 1) becomes part of the join condition

A            B            C            A            B            C           
------------ ------------ ------------ ------------ ------------ ------------
1            1            1            1            1            1           
2            2            2                                                  
3            3            3                                                  
4            4            4                                                  

4 rows fetched.

```

- "(+)" can only be converted to a left outer join or a right outer join, and cannot be converted to a full join. That is, "(+)" cannot be specified on both tables in one expression simultaneously.

```
-- Error example: Specifying "(+)" on both tables in one expression will cause an error.
SQL> SELECT * FROM t_join_1 t1, t_join_2 t2 WHERE t1.a = t2.a(+) AND t1.b(+) = t2.b; 

OG-00601, Sql syntax error: failed to generate join tree when using '(+)'
```

## Partitioned Table Data Query

When executing a SELECT query on a partitioned table, the SQL syntax is largely the same as for a regular table. However, you can leverage partitioning features to improve performance in specific ways.

### Basic Syntax

The basic syntax for querying data from partitioned tables is largely consistent with that for regular tables, except that the partition or subpartition to be queried can be specified in the FROM clause.

```
SELECT columns | expression | function
FROM [schema.]table_name [PARTITION (partition_name) | SUBPARTITION (subpartition_name)] [alias1]
[{JOIN TYPE} JOIN [Schema.]{table_name [PARTITION (partition_name) | SUBPARTITION (subpartition_name)] | view | subquery} [alias2]]
[WHERE conditions]
[GROUP BY columns]
[HAVING conditions]
[ORDER BY columns];
```

### Example

```
-- Query data from a specified partition of a partitioned table.
SELECT * FROM table_name PARTITION (partition_name) where conditions;

-- Query data from a specific subpartition of a partitioned table.
SELECT * FROM table_name SUBPARTITION (subpartition_name) WHERE conditions;

-- JOIN statement with specified partition: Query records from subpartition p_2024_q3_north in the sales table joined with the products table.
SELECT s.sale_id, p.product_name, s.amount FROM sales SUBPARTITION (p_2024_q3_north) s JOIN products p ON s.product_id = p.product_id;
```

## EXPLAIN Statement

Displays the execution plan of an SQL statement. By analyzing the EXPLAIN information of an SQL statement, you can understand the execution plan, index usage, index scan methods, table JOIN methods, and other information, thereby optimizing the performance of SQL statement execution.

### Basic Syntax

```
    EXPLAIN [PLAN FOR] SQL_QUERY;
```

- **EXPLAIN [PLAN FOR]**: The PLAN FOR field is optional and has the same functionality as the EXPLAIN statement, used to display the execution plan of an SQL statement.

- **SQL_QUERY**: The SQL statement to be analyzed. DML statements are limited to SELECT, DELETE, and UPDATE only. It must be a valid SQL statement that complies with the syntax rules of the specific statement; otherwise, an error is reported.

### Example

```

SQL> explain select * from t_join_1 t1 left outer join t_join_2 t2 on t1.a = t2.b;

EXPLAIN PLAN OUTPUT                                             
----------------------------------------------------------------
--------------------------------------------------------------------------------------
| Id | Operation               | Owner | Name        | Rows | Cost  | Bytes | Remark |
--------------------------------------------------------------------------------------
| 0  | SELECT STATEMENT        |       |             | 0    | 20000 |       |        |
| 1  |   NESTED LOOPS OUTER    |       |             | 0    | 20000 |       |        |
| 2  |       TABLE ACCESS FULL | SYS   | T_JOIN_1 T1 | 0    | 10000 |       |        |
| 3  |       TABLE ACCESS FULL | SYS   | T_JOIN_2 T2 | 0    | 10000 |       |        |
--------------------------------------------------------------------------------------
Predicate Information (identified by id):                       
-----------------------------------------                       
    3 - filter: T1.A = T2.B                                     

11 rows fetched.

SQL> EXPLAIN PLAN FOR SELECT LPAD(' ', 2 * LEVEL) || name AS employee_name, department_id
  2 FROM employees
  3 START WITH manager_id IS NULL
  4 CONNECT BY PRIOR employee_id = manager_id
  5 ORDER SIBLINGS BY name;

EXPLAIN PLAN OUTPUT                                             
----------------------------------------------------------------
-------------------------------------------------------------------------------------------
| Id | Operation                      | Owner | Name      | Rows | Cost  | Bytes | Remark |
-------------------------------------------------------------------------------------------
| 0  | SELECT STATEMENT               |       |           | 0    | 0     |       |        |
| 1  |   QUERY SORT SIBLINGS ORDER BY |       |           | 0    | 0     |       |        |
| 2  |     START WITH                 |       |           | 0    | 0     |       |        |
| 3  |         TABLE ACCESS FULL      | SYS   | EMPLOYEES | 0    | 10000 |       |        |
| 4  |     CONNECT BY                 |       |           | 0    | 0     |       |        |
| 5  |         TABLE ACCESS FULL      | SYS   | EMPLOYEES | 0    | 10000 |       |        |
-------------------------------------------------------------------------------------------
Predicate Information (identified by id):                       
-----------------------------------------                       
    2 - filter: MANAGER_ID IS NULL                              
    3 - filter: MANAGER_ID IS NULL                              
    4 - filter: PRIOR EMPLOYEE_ID = MANAGER_ID                  

15 rows fetched.
```

## Example

```sql
-- Query all employees.
SELECT * FROM employees;

-- Query a specified department and sort by salary.
SELECT name, salary FROM employees WHERE department_id = 10 ORDER BY salary DESC;

-- Use aggregate functions to calculate the average salary of each department.
SELECT department_id, AVG(salary) AS avg_salary FROM employees GROUP BY department_id;
```
