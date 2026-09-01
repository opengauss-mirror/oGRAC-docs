# ALTER SESSION

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:04:25.035Z pushedAt=2026-08-27T11:18:29.328Z -->

## Function Description

Adjusts configuration parameters of the current database session.

## Precautions

- Modifying session parameters typically requires sufficient permissions. Some parameters take effect only for the current session.

- After enabling high-risk options such as `NOLOGGING`, you should proactively trigger a checkpoint to ensure data persistence.

- Settings such as time zone and date format are valid only within the current session and revert to their default values upon disconnection.

## Syntax

```sql

ALTER SESSION
{
    SET
    {
        COMMIT_WAIT_LOGGING = { WAIT | NOWAIT }
        | COMMIT_MODE = { IMMEDIATE | BATCH }
        | _SHOW_EXPLAIN_PREDICATE = { TRUE | FALSE }
        | _OUTER_JOIN_OPTIMIZATION = { ON | OFF }
        | TIME_ZONE = '[+|-]hh:mm'
        | LOCK_WAIT_TIMEOUT = timeout
        | CURRENT_SCHEMA = schema_value
        | nls_param = nls_param_value
        | cbo_param = cbo_param_value
    }
}
|
{ ENABLE | DISABLE }
{
    TRIGGERS
    | INTERACTIVE TIMEOUT
    | NOLOGGING
    | OPTINFO_LOG
}

```

## Parameter Description

- `COMMIT_WAIT_LOGGING = { WAIT | NOWAIT }`

    Defines whether the server process committing a transaction waits for the log writer to fully write the redo information into the redo log file.<br>
    Default value: `WAIT`

  - `WAIT`

    The server process will wait for acknowledgment that the redo log write has completed. In most scenarios, this is the most reliable option and is recommended.

  - `NOWAIT`

    A transaction is committed immediately regardless of whether the redo information has been persisted to the log file. This option can improve transaction processing speed, but there is a risk of data loss due to system failure.

- `COMMIT_MODE = { IMMEDIATE | BATCH }`

    Specifies how the log writer processes redo information.<br>
    Default value: `IMMEDIATE`

  - `IMMEDIATE`

    Each transaction commit immediately triggers a log write to the disk. Setting the value to `IMMEDIATE` may affect overall transaction throughput performance due to frequent forced disk I/O operations.

  - `BATCH`

    The redo information of multiple transactions is cached in memory, and then written to the log file in batches when a certain amount is accumulated. Setting the value to `BATCH` can improve performance, but if an instance failure occurs, the redo information that has not yet been written to the log file may be lost.

- `_SHOW_EXPLAIN_PREDICATE = { TRUE | FALSE }`

    Controls whether predicate (filter condition) information is included in the execution plan output.

  - `TRUE`

    This option allows predicate conditions to be displayed in the generated execution plan.

  - `FALSE`

    This option disables the display of predicate conditions in the execution plan.

- `_OUTER_JOIN_OPTIMIZATION = { ON | OFF }`

    Controls whether the cost-based optimizer (CBO) performs outer join reordering optimization.

  - `ON`

    With this function enabled, the CBO performs reordering optimization on outer join operations to find a better execution plan.

  - `OFF`

    With this function disabled, the CBO does not perform reordering optimization on outer join operations.

- `TIME_ZONE = '[+|-]hh:mm'`

    Sets the time zone for the current session. By specifying a string in the `'[+|-]hh:mm'` format (enclosed in single quotes), you can set the session time zone offset relative to Coordinated Universal Time (UTC). `+` indicates that the time zone is ahead of UTC, and `-` indicates that the time zone is behind UTC. For example, Beijing time is `'+08:00'`.

    To query the time zone setting of the current session, run `SELECT SESSIONTIMEZONE FROM DUAL;`.

    Default value: the time zone setting of the client OS where the session is established.

    The valid time zone offset ranges from –12:00 to [+]14:00.

- `LOCK_WAIT_TIMEOUT = timeout`

    Sets the maximum time the current session is willing to wait when a requested resource is locked.

    `timeout` indicates the wait duration, in milliseconds. When the session detects that the required resource is locked, it waits for a period specified by `timeout`. If the lock is not released within this period, a lock wait timeout error is reported.

- `CURRENT_SCHEMA = schema_value`

    Sets the default schema for the current session. The default value is the schema of the logged-in user.

    - **When the current session is in the root tenant**:

        - If `schema_value` includes a tenant prefix, the session switches to a specified schema under the non-root tenant.

        - If `schema_value` has no tenant prefix, the session switches to a schema under the root tenant.

    - **When the current session is in a non-root tenant**:

        - If `schema_value` includes a tenant prefix, the prefix must match the current tenant name; otherwise, an invalid operation error is returned.

        - If `schema_value` has no tenant prefix, the session switches to a schema under the current tenant.

- `nls_param = nls_param_value`

    Sets session parameters related to National Language Support (NLS).

    - **The options for `nls_param` are as follows**:

      - `NLS_DATE_FORMAT`: The default value is `"YYYY-MM-DD HH24:MI:SS"`.

      - `NLS_TIMESTAMP_FORMAT`: The default value is `"YYYY-MM-DD HH24:MI:SS.FF"`.

      - `NLS_TIMESTAMP_TZ_FORMAT`: The default value is `"YYYY-MM-DD HH24:MI:SS.FF TZH:TZM"`.

      - `NLS_TIME_FORMAT`: The default value is `"HH:MI:SS.FF AM"`.

      - `NLS_TIME_TZ_FORMAT`: The default value is `"HH:MI:SS.FF AM TZR"`.

- `cbo_param = cbo_param_value`

    Sets the tuning parameters related to the CBO.

    The options for `cbo_param` are as follows:

    - `CBO_INDEX_CACHING`: The value is an integer ranging from 0 to 100, in percentage. The default value is `0`.

    - `CBO_INDEX_COST_ADJ`: The value is an integer ranging from 1 to 10000, in percentage. The default value is `100`.

- `{ ENABLE | DISABLE } { TRIGGERS | INTERACTIVE TIMEOUT | NOLOGGING | OPTINFO_LOG }`:

  - `TRIGGERS`

    - `ENABLE`: The SQL statements executed in the current session will activate the associated triggers.

    - `DISABLE`: The SQL statements executed in the current session will not activate triggers.

  - `INTERACTIVE TIMEOUT`

    - `ENABLE`: Enables idle session timeout detection. By default, if a session has no SQL requests for 30 consecutive minutes, the server will automatically terminate the session.

    - `DISABLE`: Disables idle session timeout detection.

  - `NOLOGGING`

    - `ENABLE`: In the current session, redo and undo logs are not recorded during insertion. **Warning**: This is a high-risk operation and is not recommended for routine use. If you must use it, ensure that after the `NOLOGGING` insertion is complete, you explicitly execute a full checkpoint to persist data to the disk before performing other normal service operations. Otherwise, if the database crashes abnormally due to a power failure or other issues, normal recovery may not be possible.

    - `DISABLE`: In the current session, redo and undo logs are recorded normally during insertion.

  - `OPTINFO_LOG`

    - `ENABLE`: Enables detailed optimizer logging for the current session. The execution plan generation details are logged to the `log/opt/zengine.opt` file. By default, the system automatically disables this logging 2 minutes after it is enabled. To continue logging after it expires, you must re-run this command to enable it again.

    - `DISABLE`: Disables detailed optimizer logging for the current session.

## Examples

- Set the session to commit transactions after redo information is written to the log file.

    ```sql
        ALTER SESSION SET COMMIT_WAIT_LOGGING = WAIT;
    ```

- Set the log writer to immediately write redo information for each transaction.

    ```sql
        ALTER SESSION SET COMMIT_MODE = IMMEDIATE;
    ```

- Set the current session time zone to UTC+8 (Beijing time).

    ```sql
        ALTER SESSION SET TIME_ZONE = '+08:00';
    ```

- Set the date display format for the session.

    ```sql
        ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD HH24:MI:SS';
    ```

- If the `user1` user exists, switch the default schema of the current session to `user1`.

    ```sql
        ALTER SESSION SET current_schema = user1;
    ```

- Enable predicate information display in the execution plan.

    ```sql
        ALTER SESSION SET _SHOW_EXPLAIN_PREDICATE = TRUE;
    ```

- Set the session lock wait timeout period to 0 milliseconds (that is, the session does not wait and returns an error immediately).

    ```sql
        ALTER SESSION SET LOCK_WAIT_TIMEOUT = 0;
    ```

- Enable triggers for the current session.

    ```sql
        ALTER SESSION ENABLE TRIGGERS;
    ```

- Disable idle session timeout detection.

    ```sql
        ALTER SESSION DISABLE INTERACTIVE TIMEOUT;
    ```

- Enable the `NOLOGGING` insertion mode for the current session.

    ```sql
        ALTER SESSION ENABLE NOLOGGING;
    ```