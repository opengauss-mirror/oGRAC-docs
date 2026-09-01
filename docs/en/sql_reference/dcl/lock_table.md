# LOCK TABLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:35:44.839Z pushedAt=2026-07-29T06:14:15.826Z -->

## Function Description

This statement is used to lock a database table.

## Precautions

-- Users executing this statement must have the `LOCK ANY TABLE` system privilege, except for the sys user and the table owner.

-- The `LOCK TABLE` command can only be executed within a transaction. When the transaction is committed or rolled back, the locked table is automatically unlocked.

## Syntax

LOCK TABLE { [ <i>schema_name</i>. ]<i>table_name</i> } [ , ... ] IN <i>lockmode</i> MODE

Where the <i>lockmode</i> clause takes the value: { SHARE | EXCLUSIVE }

## Parameter Description

- `[ schema_name. ]`: The username to which the schema belongs. If not explicitly specified, the default value is the currently logged-in user.

- `table_name`: The name of the table to be locked, which may carry a schema name prefix. The order of tables declared in the syntax is the locking order. The value range is existing table names.

- `SHARE`: A shared lock that allows concurrent DML operations such as queries, insertions, updates, and deletions, but prohibits modifications to the table structure (i.e., DDL operations). In addition to explicit locking, DML statements automatically acquire this type of lock.

- `EXCLUSIVE`: An exclusive lock that allows only concurrent queries on the target table and prohibits all other operations. In addition to explicit addition, DDL commands that modify the table structure automatically acquire this lock on user tables, and some operations also acquire it on system tables.

## Example

Lock the table salary_2025

--Drop the table salary_2025.

DROP TABLE IF EXISTS salary_2025;

--Create the table salary_2025.

CREATE TABLE salary_2025(id INT NOT NULL, name CHAR(50), job VARCHAR(30), salary NUMBER);

--Lock the table salary_2025.

LOCK TABLE salary_2025 IN SHARE MODE;

--Insert data into the table salary_2025.

INSERT INTO salary_2025(id, name, job, salary) VALUES(23,'liubei','manager',10000);

-- Insert data into the salary_2025 table.

INSERT INTO salary_2025(id, name, job, salary) VALUES(24,'guanyu','tester',8000);

-- Insert data into the salary_2025 table.

INSERT INTO salary_2025(id, name, job, salary) VALUES(25,'zhangfei','developer',8000);

-- Commit.

COMMIT;
