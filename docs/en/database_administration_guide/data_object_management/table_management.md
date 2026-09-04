# Table Management

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:00:47.510Z pushedAt=2026-08-20T06:11:23.436Z -->

A table (`TABLE`) is the most basic logical unit for data storage in a database. It organizes and stores structured data in the form of rows and columns. The objects directly operated on by user data manipulation operations ([INSERT](../../sql_reference/dml/insert.md), [DELETE](../../sql_reference/dml/delete.md), [UPDATE](../../sql_reference/dml/update.md), and [SELECT](../../sql_reference/select.md)) are tables. When creating a table, `TABLE_NAME`, `COLUMN_NAME`, and the corresponding `COLUMN_TYPE` must exist. In addition, each column attribute can specify additional column constraints (such as `DEFAULT`, `NOT NULL`, `UNIQUE`, etc.).

## Creating a Regular Table

Create a regular table `EMPLOYEES` in the tablespace `OGRACSPACE`.

```sql
CREATE TABLE [Schema.]TABLE_NAME
(
    COLUMN_NAME1 COLUMN_TYPE [DEFAULT DEFAULT_VALUE] [{NOT | DEFAULT} NULL],
    COLUMN_NAME2 COLUMN_TYPE ...
) [TABLESPACE TABLESPACE_NAME];

-- Example
CREATE TABLE EMPLOYEES
(
    ID INT NOT NULL,
    NAME VARCHAR(10) DEFAULT NULL,
    SALARY DECIMAL(10,2),
    HIRE_DATE DATE
) TABLESPACE OGRACSPACE;
```

## Altering a Table Structure

### Renaming a Table

```sql
ALTER TABLE TABLE_NAME RENAME TO TABLE_NAME_NEW;
```

### Adding, Deleting, and Modifying Columns

```sql
-- Add a column
ALTER TABLE TABLE_NAME ADD COLUMN_NAME COLUMN_TYPE;
-- Delete a column
ALTER TABLE TABLE_NAME DROP COLUMN_NAME;
-- Modify the column type
ALTER TABLE TABLE_NAME MODIFY COLUMN_NAME COLUMN_TYPE;
```

### Adding, Deleting, and Modifying Column Constraints

```sql
-- Add a primary key constraint to the ID column of table EMPLOYEES
ALTER TABLE EMPLOYEES ADD CONSTRAINT PRIMARY_KEY1 PRIMARY KEY (ID);
-- Add a unique constraint to the NAME column of table EMPLOYEES
ALTER TABLE EMPLOYEES ADD CONSTRAINT UNIQUE_NAME1 UNIQUE (NAME);
-- Add a range check constraint to the SALARY column of table EMPLOYEES
ALTER TABLE EMPLOYEES ADD CONSTRAINT SALARY_MIN_CHECK CHECK (SALARY > 12345);

-- Rename constraints
ALTER TABLE EMPLOYEES RENAME CONSTRAINT UNIQUE_NAME1 TO UNIQUE_NAME2;
ALTER TABLE EMPLOYEES RENAME CONSTRAINT PRIMARY_KEY1 TO PRIMARY_KEY2;

-- Delete the primary key constraint
ALTER TABLE EMPLOYEES DROP CONSTRAINT PRIMARY_KEY2;
-- Delete the SALARY_MIN_CHECK constraint
ALTER TABLE EMPLOYEES DROP CONSTRAINT SALARY_MIN_CHECK;
```

### Enabling and Disabling Constraints

Users can use `ALTER TABLE ... DISABLE|ENABLE CONSTRAINT_NAME` to enable or disable constraints defined on table columns.

```sql
ALTER TABLE EMPLOYEES DISABLE CONSTRAINT UNIQUE_NAME1;
ALTER TABLE EMPLOYEES ENABLE CONSTRAINT UNIQUE_NAME1;
```

### Foreign Key Information

A foreign key (`Foreign Key`) is a column (or a set of columns) in one table that references the primary key (`Primary Key`) or unique key (`Unique Key`) of another table. It is used to establish a link between the two tables.

```sql
-- Create tables
DROP TABLE IF EXISTS departments;
DROP TABLE IF EXISTS students;

-- dept_id in the students table is the primary key of departments; dept_id can serve as the foreign key of the students table
CREATE TABLE departments (
    dept_id   INT PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name       VARCHAR(50) NOT NULL,
    dept_id    INT NOT NULL
);

-- Create foreign keys for the teachers and students tables, whose parent table is departments
ALTER TABLE students ADD CONSTRAINT FOR_KEY_1 FOREIGN KEY (dept_id) REFERENCES departments(dept_id);

-- Rename the foreign key
ALTER TABLE students RENAME CONSTRAINT FOR_KEY_1 TO FOR_KEY_ST;

-- Drop the foreign key
ALTER TABLE students DROP CONSTRAINT FOR_KEY_ST;
```

### Viewing Foreign Key Information

```sql
-- Users can use DESC to view table column information.
-- DESC TABLE_NAME
-- Example
SQL> DESC STUDENTS

Name                                Null?    Type                                
----------------------------------- -------- ------------------------------------
STUDENT_ID                          NOT NULL BINARY_BIGINT                       
NAME                                NOT NULL VARCHAR(50 BYTE)                    
DEPT_ID                             NOT NULL BINARY_INTEGER                      

-- Users can use the SHOW CREATE command to view detailed table information.
-- SHOW CREATE TABLE TABLE_NAME
-- Example
SQL> SHOW CREATE TABLE STUDENTS

CREATE TABLE "STUDENTS"
(
  "STUDENT_ID" BINARY_BIGINT NOT NULL,
  "NAME" VARCHAR(50 BYTE) NOT NULL,
  "DEPT_ID" BINARY_INTEGER NOT NULL
)
TABLESPACE "SYSTEM"
INITRANS 2
MAXTRANS 255
PCTFREE 8
FORMAT ASF;
ALTER TABLE "STUDENTS" MODIFY "STUDENT_ID" AUTO_INCREMENT;
ALTER TABLE "STUDENTS" AUTO_INCREMENT = 0;
ALTER TABLE "STUDENTS" ADD PRIMARY KEY("STUDENT_ID");
ALTER TABLE "STUDENTS" ADD CONSTRAINT "FOR_KEY_ST" FOREIGN KEY("DEPT_ID") REFERENCES "DEPARTMENTS"("DEPT_ID");
```

## Clearing Table Data and Dropping Tables

Clearing table data means deleting all row data in the table while retaining only the structural information defined by the table. You can clear table data using `TRUNCATE` or `DELETE`:

```sql
DELETE FROM STUDENTS;
DELETE FROM STUDENTS WHERE STUDENT_ID = 1234;
COMMIT; -- Can be rolled back using ROLLBACK

TRUNCATE TABLE STUDENTS;
```

The difference between the two is that `DELETE` can delete rows that meet the filter conditions through the `WHERE` clause, and is governed by transactions, so it can be rolled back using `ROLLBACK`; the `TRUNCATE` command cannot use the `WHERE` clause and cannot be rolled back, so use it with caution.

You can drop a table using the `DROP TABLE` command. After the table is dropped, the table data is also deleted, and `ROLLBACK` is not supported.

```sql
DROP TABLE STUDENTS;
DROP TABLE IF EXISTS STUDENTS;
```
