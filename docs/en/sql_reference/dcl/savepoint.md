# SAVEPOINT

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:21.234Z pushedAt=2026-07-29T06:14:15.835Z -->

## Description

SAVEPOINT is a key feature in database transaction management. It allows multiple rollback markers to be set within a transaction, enabling partial rollback of the transaction. Its core value lies in enhancing transaction flexibility and fault tolerance.

## Notes

When rolling back to a savepoint, the database discards all operations after that savepoint, but retains the modifications made before that savepoint.

## Syntax

SAVEPOINT savepoint_name

## Parameter Description

- savepoint_name: Name of the savepoint.

## Examples

Roll back a transaction to a savepoint

--Delete the table reward_2025.

DROP TABLE IF EXISTS reward_2025;

--Create the table reward_2025.

CREATE TABLE reward_2025(employee_id INT NOT NULL, employee_name CHAR(50), money_award NUMBER);

--Write data 1.

INSERT INTO reward_2025(employee_id, employee_name, money_award) VALUES(1, 'zhangsan', 1200);

--Set savepoint 1.

SAVEPOINT sn1;

--Insert data 2.

INSERT INTO reward_2025(employee_id, employee_name, money_award) VALUES(2, 'lisi', 2200);

--Set savepoint 2.

SAVEPOINT sn2;

--Query data from the reward_2025 table.

SELECT * FROM reward_2025;

--Roll back to savepoint 1.

ROLLBACK TO SAVEPOINT sn1;

--Query the data in the reward_2025 table.

SELECT * FROM reward_2025;

Only the data written before savepoint 1 is expected to be found:

EMPLOYEE_ID  EMPLOYEE_NAME                                      MONEY_AWARD
------------ -------------------------------------------------- ----------------------------------------
1            zhangsan                                           1200

1 rows fetched.
