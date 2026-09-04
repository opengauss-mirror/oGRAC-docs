# SHUTDOWN

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:38.941Z pushedAt=2026-07-29T06:14:15.842Z -->

## Description

The database provides a dedicated SQL statement for graceful shutdown: when the `SHUTDOWN` command is executed, the system first stops TCP listening, waits for the transactions of all sessions to complete, and then terminates the main process.

## Precautions

-- If the `SHUTDOWN` command is executed while a transaction is in progress, the system will report an error.

-- When neither the `IMMEDIATE` nor `ABORT` parameter is specified, the normal mode is used by default. In this mode, executing `SHUTDOWN` stops the system from accepting client requests, waits for unfinished transactions to complete and all connected sessions to disconnect, and then shuts down.

-- When `SHUTDOWN` is executed in normal or immediate mode, the database saves statistical information and flushes it to the SYS_DML_STATS system table before shutting down.

## Syntax

SHUTDOWN [ IMMEDIATE | ABORT ]

## Parameter Description

- `IMMEDIATE`: Stops accepting client requests, rolls back all unfinished transactions, and then exits the main process.

- `ABORT`: Stops accepting client requests, and the main process exits immediately (without waiting for transaction processing).

## Examples

-- Execute the following statement to immediately stop the database instance.

SHUTDOWN ABORT;
