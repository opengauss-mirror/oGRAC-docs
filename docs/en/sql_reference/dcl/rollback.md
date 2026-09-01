# ROLLBACK

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:15.786Z pushedAt=2026-07-29T06:14:15.832Z -->

## Description

This statement is used to undo all operations in the current transaction and terminate the transaction.

## Notes

It is advised that users explicitly confirm the transaction status by using the `COMMIT` or `ROLLBACK` command before exiting a program. If no commit operation is actively performed and the program terminates abnormally, the database will automatically undo all unsaved modifications to ensure data consistency.

For DML transaction operations, an explicit commit or rollback is required for the transaction to be committed or rolled back. There is one exception: if a DDL operation is present, all previously uncommitted DML operations will be automatically committed when a DDL statement is executed. In addition, if a failure occurs during DML operations, it does not affect the continued execution of subsequent DML operations or the transaction rollback operation.

After a DDL statement, the preceding DML operations cannot be rolled back. For DDL operations, a successful execution is automatically committed, and a failed execution is automatically rolled back. Users do not need to explicitly commit or roll back, and a successfully executed DDL operation cannot be rolled back.

Rollback is not supported for the `CREATE TABLESPACE` and `ALTER TABLESPACE DDL` statements.

## Syntax

ROLLBACK [ TRANSACTION | TO SAVEPOINT savepoint_name ]

## Parameter Description

- `TRANSACTION`: Explicitly rolls back the transaction.

- `TO SAVEPOINT`: Rolls back to a savepoint.

- `savepoint_name`: The name of the savepoint.

## Examples

Create the table topics, write data, then undo all operations and terminate the transaction.

-- Delete the table topics.

DROP TABLE IF EXISTS topics;

-- Create the table topics.

CREATE TABLE topics(topic_id CHAR(2) NOT NULL, topic_name CHAR(16) NOT NULL, attendees INT);

-- Write data 1 into the topics table.

INSERT INTO topics(topic_id, topic_name, attendees) VALUES('0A', 'peace', 100);

-- Write data 2 into the topics table.

INSERT INTO topics(topic_id, topic_name, attendees) VALUES('0B', 'develop', 200);

--Roll back the transaction.

ROLLBACK;
