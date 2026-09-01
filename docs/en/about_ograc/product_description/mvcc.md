# MVCC

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:45:53.190Z pushedAt=2026-08-17T00:46:29.620Z -->

## Overview

oGRAC significantly enhances its high-concurrency processing capabilities through a synergistic combination of the locking mechanism, multi-version concurrency control (MVCC), and database isolation levels. Among these, MVCC is one of the core technologies for achieving high-performance concurrent access. Its fundamental idea is to resolve read-write contention with minimal or no reliance on traditional locks, thereby realizing an ideal concurrency model in which reads do not block writes, and writes do not block reads.

## Core Idea

- Data versioning: Each row of data can have multiple historical versions, maintained by `Undo` records. The data version visible to each transaction depends on the `SCN`.
- Consistent read: When a transaction reads data, it does not retrieve the latest version of the data. Instead, it retrieves a historical version that satisfies visibility criteria, as implemented by the MVCC mechanism. This process requires no locking and does not block write operations of other transactions. It applies to `SELECT` queries.
- Current read: A transaction reads the most up-to-date version of the data. This typically occurs in modification operations such as `UPDATE`, `DELETE`, or `SELECT FOR UPDATE`. A current read performs visibility checks and may apply row-level locks to ensure operational safety.
- Lock contention avoidance via MVCC: With multi-version control, most read-only transactions are completed via consistent reads, thereby avoiding lock waits with write transactions and significantly improving concurrency performance.

## MVCC Principles

### SCN

The MVCC implementation of oGRAC is based on the 64-bit timestamp `SCN` and the `Undo` logging mechanism, constructing an efficient row-level multi-version management system.

An SCN is a globally monotonically increasing timestamp in oGRAC, used to identify the chronological order of transactions. It is a 64-bit integer with the following structure:

```text
{second[32bit]} {micro-second[20bit]} {sequence[12bit]}
```

`second` and `micro-second` provide a precise time reference, while `sequence` distinguishes multiple transactions within the same microsecond to ensure uniqueness.

### Multi-Version Storage and Visibility Determination

- "Multi-version" refers to the fact that oGRAC simultaneously materializes and maintains multiple versions of data. Historical versions are stored separately from the latest version. Historical versions are retained in the undo page and linked in a chain, forming a version chain arranged in descending order of `SCN`. The current version resides in data page.
- When a transaction performs a consistent read, if the query transaction's `Query SCN` is greater than `Commit SCN`, the row version has not been modified by subsequent active transactions and is visible to the current transaction. If the version on the current data page is not visible, the system traces upward along the `Undo` chain until it finds the first historical version that satisfies the condition.

## Row Lock Mechanism

Although MVCC resolves most read-write contentions, write-write contentions still require a locking mechanism to ensure data consistency. oGRAC uses the Interested Transaction List (ITL) to implement efficient row-level lock management.

### ITL Structure and Functions

The ITL is not a centralized lock table, but rather a dynamic array embedded at the end of each data page. The header of each row record on the page contains an ITL pointer, which is used to record information about the rows currently locked by transactions on that page. Each ITL entry includes the transaction ID, lock type, transaction active status, rollback segment pointer, and other details. By default, two ITL entries exist on a page (based on the assumption that the probability of more than two concurrent transactions on the same page is low). When the number of concurrent transactions on the same page increases, the ITL can be expanded, supporting a maximum of 255 entries. If the number exceeds 255, waiting is required. Once expanded, the ITL space is retained on the page for subsequent use, thereby avoiding frequent allocation overhead.

### Row Lock Acquisition Process

When a transaction attempts to perform a current read or update operation on a row, the following steps are executed:

Check whether the row is already locked: examine whether the Row Header points to an active ITL entry. If so and the transaction is still running, wait for that transaction to commit or roll back. If no conflict exists, request an ITL entry on the current page, set the Row Header of the target row, and begin the modification operation. The old version is pushed into undo, and the new version is written to the data page. The ITL lock is released when the transaction commits or rolls back.

## Mechanism Advantages

- High-concurrency read/write performance: Most `SELECT` operations are completed through consistent reads without the need for locking, completely eliminating read-write mutual exclusion and significantly improving concurrency.
- Low row lock maintenance overhead: The ITL-based row lock mechanism localizes lock information within data pages, reducing the complexity of global lock management.
- Efficient storage structure: Data pages retain only the latest version, improving cache hit ratio and I/O efficiency; historical versions are loaded from undo on demand, saving memory usage.
- Automatic version cleanup and reclamation: Undo tablespaces support automatic reclamation, where expired versions are asynchronously reclaimed after no active transactions reference them, preventing unbounded expansion.
- Good architectural continuity: The SCN+Undo-based MVCC scheme is highly compatible and easily extensible.
