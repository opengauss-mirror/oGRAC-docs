# SET TRANSACTION

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:42.208Z pushedAt=2026-07-29T06:14:15.845Z -->

## Description

This statement is used to configure the transaction isolation level. The transaction isolation level defines the visibility of the current transaction's behavior to other concurrently executing transactions. Users can adjust the isolation level to increase visibility into other uncommitted transactions, thereby improving the concurrent processing capability of the database.

## Precautions

The session-level transaction isolation level can only be configured before a transaction starts. Modifying the isolation level during transaction execution is not allowed.

## Syntax

-- Session-level
SET TRANSACTION ISOLATION LEVEL { SERIALIZABLE | READ COMMITTED | CURRENT COMMITTED }

-- Global
ALTER SYSTEM SET DB_ISOLEVEL={ RC | CC }
SHOW PARAMETER DB_ISOLEVEL

## Parameter Description

- Session scope

  - `SERIALIZABLE`: The highest level of transaction isolation, which achieves complete isolation between transactions.

  - `READ COMMITTED`: The default isolation level, which ensures that dirty data is never read; data read within a single SQL statement is a snapshot of the same version.

  - `CURRENT COMMITTED`: In the data read within a single SQL statement, each row is the latest committed data at the moment of reading, and the read result is no longer snapshot data.

- Global scope

  - `DB_ISOLEVEL`: Configures the transaction isolation level to ensure that uncommitted dirty data is not read.

  - `RC`: Read Committed, where data read within a single SQL statement is snapshot data of the same version. This is the default isolation level.

  - `CC`: Current Committed, where each row read within a single SQL statement is the latest committed data at the moment of reading, and the read result is no longer snapshot data.

## Examples

### Configuring the Session Transaction Isolation Level

-- Drop the table.

DROP TABLE IF EXISTS salary_2025;

-- Create the table.

CREATE TABLE salary_2025(id INT NOT NULL, name CHAR(50), job VARCHAR(30), salary NUMBER);

-- Set the isolation level to READ COMMITTED before the transaction starts.

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Execute statement 1 within the transaction.

INSERT INTO salary_2025(id, name, job, salary) VALUES(23,'liubei','manager',10000);

-- Execute statement 2 within the transaction.

INSERT INTO salary_2025(id, name, job, salary) VALUES(24,'guanyu','tester',8000);

-- Execute statement 3 within the transaction.

INSERT INTO salary_2025(id, name, job, salary) VALUES(25,'zhangfei','developer',8000);

-- Commit the entire transaction.

COMMIT;

### Configuring the Global Transaction Isolation Level

--Query the global isolation level in the transaction.

SHOW PARAMETER DB_ISOLEVEL;

--Set the global isolation level of the transaction to READ COMMITTED.

ALTER SYSTEM SET DB_ISOLEVEL= RC;

-- Execute statement 1 within the transaction.

DROP TABLE IF EXISTS salary_2025;

-- Execute statement 2 within the transaction.

CREATE TABLE salary_2025(id INT NOT NULL, name CHAR(50), job VARCHAR(30), salary NUMBER);

-- Execute statement 3 within the transaction.

INSERT INTO salary_2025(id, name, job, salary) VALUES(23,'liubei','manager',10000);

-- Commit the entire transaction.

COMMIT;
