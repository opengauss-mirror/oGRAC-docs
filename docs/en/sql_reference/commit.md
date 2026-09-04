# COMMIT

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:14:31.856Z pushedAt=2026-08-13T07:12:35.741Z -->

## Function Description

Commits the current transaction, making all data operations in the transaction permanently effective and ending the transaction.

## Precautions

Data operations (`SELECT`, `DELETE`, `UPDATE`) are not automatically committed by default. When a session exits, you must explicitly execute `COMMIT`; otherwise, all uncommitted changes will be lost.
The transaction commit mechanism is described as follows:

1. For DML transaction operations, you must explicitly execute `COMMIT` or `ROLLBACK` to complete the commit or rollback. **Special case**: If a DDL statement is executed within a transaction, all previously uncommitted DML operations are automatically committed when the DDL statement is executed. In addition, if a DML operation partially fails during execution, it does not affect the execution of subsequent DML operations or the transaction commit operation.

2. After a DDL statement is executed, the previous DML operations cannot be rolled back.

Meanwhile, the DDL operation itself is automatically committed upon successful execution and automatically rolled back upon failure, without requiring the user to explicitly commit/roll back, and it cannot be rolled back after successful execution.

## Syntax

```sql
COMMIT [ TRANSACTION ]
```

## Parameter Description

- **COMMIT [ TRANSACTION ]**: Commits the current transaction. `TRANSACTION` is an optional keyword for enhanced readability, and has the same effect as executing `COMMIT` directly.

## Examples

Create the training table, insert data and update data, and then commit the operation to end the transaction.

```
--Delete the training table.
DROP TABLE IF EXISTS training;

--Create the training table.
CREATE TABLE training(staff_id INT NOT NULL, staff_name VARCHAR(16), course_name CHAR(20), course_start_date DATETIME, course_end_date DATETIME, exam_date DATETIME, score INT);

--Insert the first record into the table.
INSERT INTO training(staff_id,staff_name,course_name,course_start_date,course_end_date,exam_date,score) VALUES(10,'zhangsan','cpp','2025-11-23 12:00:00','2025-11-24 12:00:00','2025-11-25 12:00:00',72);

--Insert the second record into the table.
INSERT INTO training(staff_id,staff_name,course_name,course_start_date,course_end_date,exam_date,score) VALUES(11,'lisi','cpp','2025-11-26 12:00:00','2025-11-27 12:00:00','2025-11-28 12:00:00',87);

--Update the staff_name and course_name fields in the second record.
UPDATE training SET staff_name='lisi', course_name='INFORMATION SAFETY' WHERE staff_id=11;

--Commit the transaction.
COMMIT;
```
