# TRIGGER

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:11.699Z pushedAt=2026-07-30T08:45:37.914Z -->

## Description

A trigger is a special stored procedure associated with a table or view. It is automatically executed when a specific DML (`INSERT`, `UPDATE`, `DELETE`) operation is performed on the associated table or view. `oGRAC` triggers support three trigger timings: `BEFORE`, `AFTER`, and `INSTEAD OF`, as well as two trigger levels: statement-level and row-level.

## Notes

- To create or drop a trigger, you need the **CREATE TRIGGER** system privilege. To create a trigger on an object in another user's schema, you also need the corresponding object privilege.

- Triggers cannot be created on objects owned by **SYS**.

- The maximum number of triggers per table is **8**.

- The data type of the "OF col" column cannot be `LOB`.

- Triggers cannot be created on local temporary tables (whose names start with `#`).

- Executing certain SQL statements within the trigger body against the base table being modified by DML may cause a mutating table error (`ERR_TAB_MUTATING`).

- DDL and DCL statements are not allowed inside a trigger body, and ordinary users cannot create objects for system users.

- Each trigger object can be bound to only one timing (either statement-level or row-level). Compound triggers (`COMPOUND TRIGGER`) are not supported.

- When creating a trigger, the trailing "/" symbol must be added as the terminator; otherwise, an error will be reported.

## Syntax Format

### Trigger Creation

```sql
CREATE [OR REPLACE] TRIGGER [IF NOT EXISTS] [schema.]trigger_name
    { BEFORE | AFTER | INSTEAD OF }
    { INSERT | DELETE | UPDATE [OF col1 [, col2 ...]] }
    [ OR { INSERT | DELETE | UPDATE [OF ...] } ... ]
    ON [schema.]table_or_view
    [ FOR EACH ROW ]
    [ DECLARE
        ...
      ]
    BEGIN
        ...
    END;
/
```

> **NOTE**

- **CREATE [OR REPLACE] TRIGGER [IF NOT EXISTS] [schema.]trigger_name**

  Creates or replaces a trigger. `OR REPLACE` indicates that if a trigger with the same name already exists, it will be overwritten; `IF NOT EXISTS` indicates that if a trigger already exists, no error is reported and no creation is performed.

- **[schema.]**

  Optional. The schema to which the trigger belongs; if omitted, the current session schema is used.

- **trigger_name**

  The trigger name, which must comply with object naming rules.

- **{ BEFORE | AFTER | INSTEAD OF }**

  The trigger timing. Choose one of the three options, placed immediately after the trigger name.

  - **BEFORE**: Executes before the DML actually modifies the data.

  - **AFTER**: Executes after the DML modifies the data.

  - **INSTEAD OF**: Replaces the original DML execution on the view. It can only be created on views and cannot be used on regular tables.

- **{ INSERT | DELETE | UPDATE [OF col1 [, col2 ...]] }**

  The trigger event, specifying under which DML operation the trigger is activated.

  - **INSERT**: Triggered when a row is inserted.

  - **DELETE**: Triggered when a row is deleted.

  - **UPDATE**: Triggered when a row is updated.

  - **UPDATE OF** ***col1*** **[,** ***col2*** **...]**

    Triggered only when the listed columns are modified in the `SET` clause; the columns must exist in the base table, be visible, and cannot be of `LOB` type.

- **[ OR { INSERT | DELETE | UPDATE [OF ...] } ... ]**

  Optional. Use `OR` to connect multiple events so that the same trigger responds to multiple DML operations, for example, `INSERT OR UPDATE ON tbl`.

- **ON [schema.]table_or_view**

  The table or view to which the trigger is bound.

  - Regular tables, unlogged tables, and transaction-level/session-level temporary tables: only **BEFORE** / **AFTER** are supported.

  - Views: only **INSTEAD OF** is supported.

  - Local temporary tables (starting with `#`): creating triggers is not supported.

- **[ FOR EACH ROW ]**

  Optional. Specifies a **row-level** trigger; if omitted, the trigger is **statement-level**.

  - Row-level: the trigger body is executed once for each affected row; `:NEW` and `:OLD` can be used.

  - Statement-level: Executes only once per DML statement; `:NEW` and `:OLD` cannot be used.

  - `BEFORE` / `AFTER` row-level triggers must explicitly specify `FOR EACH ROW`.

  - `INSTEAD OF` triggers are typically row-level and also use `FOR EACH ROW` in their syntax.

- **[ DECLARE ... ]**

  Optional. The declaration area for trigger local variables, cursors, and so on, with syntax identical to that of a PL/SQL block.

- **BEGIN ... END**

  The trigger execution body, consisting of a sequence of PL/SQL statements. Statements must be submitted with a terminating `/` (consistent with SQL*Plus conventions).

Example:

- Create a table.

    ```sql
    DROP TABLE IF EXISTS test;
    CREATE TABLE test (
        emp_id   NUMBER PRIMARY KEY,
        emp_name VARCHAR2(50),
        job      VARCHAR2(50),
        salary   NUMBER(10, 2)
    );
    ```

- **test_trigger_br**: A row-level `BEFORE INSERT` trigger that converts `emp_name` and `job` to uppercase before each row is inserted.

    ```sql
    DROP TRIGGER IF EXISTS test_trigger_br;
    CREATE OR REPLACE TRIGGER test_trigger_br
    BEFORE INSERT ON test
    FOR EACH ROW
    BEGIN
        :NEW.emp_name := UPPER(:NEW.emp_name);
        :NEW.job      := UPPER(:NEW.job);
    END;
    /
    ```

- **test_trigger_bs**: A statement-level `BEFORE INSERT` trigger that assigns a value to a local variable before each INSERT begins.

    ```sql
    DROP TRIGGER IF EXISTS test_trigger_bs;
    CREATE OR REPLACE TRIGGER test_trigger_bs
    BEFORE INSERT ON test
    DECLARE
        v_cnt NUMBER := 0;
    BEGIN
        v_cnt := v_cnt + 1;
    END;
    /
    ```

- **tri_sal_audit**: A row-level `AFTER UPDATE OF salary` trigger that executes after a row is written only when the `salary` column is updated (the body is empty as a placeholder).

    ```sql
    DROP TRIGGER IF EXISTS tri_sal_audit;
    CREATE OR REPLACE TRIGGER tri_sal_audit
    AFTER UPDATE OF salary ON test
    FOR EACH ROW
    BEGIN
        NULL;
    END;
    /
    ```

- Create the view **v_emp**, and then create **tri_v_emp_io**: an `INSTEAD OF INSERT` trigger that converts an insert into the view into an insert into the base table `test`.

    ```sql
    CREATE OR REPLACE VIEW v_emp AS
    SELECT emp_id, emp_name, job, salary FROM test;

    DROP TRIGGER IF EXISTS tri_v_emp_io;
    CREATE OR REPLACE TRIGGER tri_v_emp_io
    INSTEAD OF INSERT ON v_emp
    FOR EACH ROW
    BEGIN
        INSERT INTO test (emp_id, emp_name, job, salary)
        VALUES (:NEW.emp_id, :NEW.emp_name, :NEW.job, :NEW.salary);
    END;
    /
    ```

## Dropping a Trigger

```sql
DROP TRIGGER [IF EXISTS] [schema.]trigger_name;
```

> **NOTE**

- **DROP TRIGGER [IF EXISTS] [schema.]trigger_name**

  Drops the specified trigger.

- **[IF EXISTS]**

  Optional. No error is reported if the trigger does not exist.

- **[schema.]**

  Optional. The owner of the trigger. If omitted, the current schema is used.

- ***trigger_name***

  The name of the trigger to be dropped.

Example:

- Drop the trigger.

    ```sql
    DROP TRIGGER test_trigger_br;
    ```

- No error is reported when the trigger does not exist.

    ```sql
    DROP TRIGGER IF EXISTS test_trigger_br;
    ```

## Altering a Trigger

```
ALTER TRIGGER [schema.]trigger_name ENABLE;
ALTER TRIGGER [schema.]trigger_name DISABLE;
```

> **NOTE**

- **ENABLE**

  Enables a disabled trigger, after which DML operations will execute the trigger again according to its type and timing.

- **DISABLE**

Disables the trigger, after which DML operations skip the trigger and do not execute its trigger body.

- **[schema.]**

  Optional. The owner of the trigger.

- ***trigger_name***

  The name of the trigger to enable or disable.

Example:

- The prerequisites are the same as those in the "Trigger Creation" example above, including creating the table and the **test_trigger_br** trigger.

- Disable the trigger (after this, DML operations on the table will no longer execute the trigger).

    ```sql
    ALTER TRIGGER test_trigger_br DISABLE;
    ```

- Re-enable the trigger.

    ```sql
    ALTER TRIGGER test_trigger_br ENABLE;
    ```

## Row-Level Pseudocolumns :NEW / :OLD

In a row-level trigger or an `INSTEAD OF` trigger, pseudocolumns are used to access the old and new values of the current row (they are not real columns of the table).

### Syntax Format

Access operation table columns:

```
{ :OLD | :NEW }.col
```

> **NOTE**

- **col**:

  Column name, consistent with the case sensitivity of the database.

Access pseudocolumns:

```
{ :OLD | :NEW }.{ rowid | rowscn }
```

### Access Rules

- **:NEW.column_name**

  The value of each column in the new row.

  - Applicable to: **BEFORE INSERT**, **BEFORE UPDATE**, **INSTEAD OF** row-level triggers (read-only or assignment).

  - Assigning a value to `:NEW.col` in **BEFORE INSERT** / **BEFORE UPDATE** can change the data to be written.

  - In an **AFTER** trigger, `:NEW` is generally read-only and cannot be modified.

  - Quoted column names are supported, for example, `:NEW."ColName"`.

- **:OLD.column_name**

  The value of each column in the old row.

  - Applicable to: **BEFORE/AFTER UPDATE**, **BEFORE/AFTER DELETE**, and **INSTEAD OF** row-level triggers.

  - **Not allowed** to assign a value to `:OLD.col`.

- **:NEW.rowid** / **:OLD.rowid**

  The row ID of the current row, of string type.

- **:NEW.rowscn** / **:OLD.rowscn**

  SCN-related information of the current row, of big integer type.

**Restrictions:**

- Pseudo columns can only appear in row-level or `INSTEAD OF` triggers; using them in statement-level triggers will cause an error.

- Modification of `:NEW` is only allowed in **BEFORE INSERT** and **BEFORE UPDATE** row-level triggers.

Example:

- Prerequisites (create a test table and insert data):

    ```sql
    DROP TRIGGER IF EXISTS TEST_TRIGGER_COL;
    DROP TRIGGER IF EXISTS TEST_TRIGGER_PSEUDO;
    DROP TABLE IF EXISTS TEST_TRIGGER;

    CREATE TABLE TEST_TRIGGER(A INT, B INT);
    INSERT INTO TEST_TRIGGER VALUES(1, 2);
    ```

- Example of using trigger table columns (`:OLD.col`, `:NEW.col`):

    ```sql
    --Create trigger TEST_TRIGGER_COL (row-level AFTER UPDATE, print table columns :OLD.A and :NEW.A).
    DROP TRIGGER IF EXISTS TEST_TRIGGER_COL;
    CREATE OR REPLACE TRIGGER TEST_TRIGGER_COL
    AFTER UPDATE ON TEST_TRIGGER
    FOR EACH ROW
    IS
    BEGIN
        DBE_OUTPUT.PRINT_LINE(':OLD.A = ' || :OLD.A);
        DBE_OUTPUT.PRINT_LINE(':NEW.A = ' || :NEW.A);
    END;
    /
    ```

- Example of using trigger pseudocolumns (`:OLD.rowid`, `:OLD.rowscn`, `:NEW.rowid`, `:NEW.rowscn`):

    ```sql
    --Create trigger TEST_TRIGGER_PSEUDO (row-level AFTER UPDATE, print rowid/rowscn pseudocolumns).
    DROP TRIGGER IF EXISTS TEST_TRIGGER_PSEUDO;
    CREATE OR REPLACE TRIGGER TEST_TRIGGER_PSEUDO
    AFTER UPDATE ON TEST_TRIGGER
    FOR EACH ROW
    IS
    BEGIN
        DBE_OUTPUT.PRINT_LINE(':OLD.rowid' || :OLD.rowid);
        DBE_OUTPUT.PRINT_LINE(':OLD.rowscn' || :OLD.rowscn);
        DBE_OUTPUT.PRINT_LINE(':NEW.rowid' || :NEW.rowid);
        DBE_OUTPUT.PRINT_LINE(':NEW.rowscn' || :NEW.rowscn);
    END;
    /
    ```

- Execute and view the results:

    ```sql
    -- Enable server output (otherwise DBE_OUTPUT.PRINT_LINE will not be displayed on the client).
    SET SERVEROUTPUT ON;

    -- Update table TEST_TRIGGER, the trigger prints the values of column A before and after the update, along with rowid/rowscn.
    UPDATE TEST_TRIGGER SET A = 10 WHERE A = 1;
    ```

    Table column trigger output:

    ```
    :OLD.A = 1
    :NEW.A = 10

    1 rows affected.
    ```

    Pseudocolumn trigger output:

    ```
    :OLD.rowid000000000271400000
    :OLD.rowscn8923265734799361
    :NEW.rowid000000000271400000
    :NEW.rowscn8923265734799361

    1 rows affected.
    ```

    If `0 rows affected` appears, it means there is no row with `A = 1` (please first confirm that `INSERT` has been executed and the data is still in the table).

## Complete Flow Example (Table-Based DML Trigger)

The following example uses table `t1` (to receive inserted data) and table `tg_log` (to record the trigger execution trace) to demonstrate the trigger order of the four timing points in an **INSERT** scenario, and completes the creation, verification, and deletion of triggers. Each trigger inserts a description line (such as `before insert`) into `tg_log` for easy comparison of the execution order.

- Preparing tables

  ```sql
  DROP TABLE IF EXISTS tg_log;
  DROP TABLE IF EXISTS t1;
  DROP TRIGGER IF EXISTS tg_bi_s;
  DROP TRIGGER IF EXISTS tg_bi_r;
  DROP TRIGGER IF EXISTS tg_ai_r;
  DROP TRIGGER IF EXISTS tg_ai_s;

  CREATE TABLE t1 (
      id  NUMBER PRIMARY KEY,
      val VARCHAR2(10)
  );

  CREATE TABLE tg_log (
      seq NUMBER,
      msg VARCHAR2(32)
  );
  ```

- Creating triggers

  **tg_bi_s**: A statement-level `BEFORE INSERT` trigger that inserts a `before insert` record into `tg_log` before the entire `INSERT` statement begins.

    ```sql
    CREATE OR REPLACE TRIGGER tg_bi_s
    BEFORE INSERT ON t1
    BEGIN
        INSERT INTO tg_log (seq, msg)
        SELECT NVL(MAX(seq), 0) + 1, 'before insert' FROM tg_log;
    END;
    /
    ```

**tg_bi_r**: row-level `BEFORE INSERT`, adds the `R_` prefix to `val` before each row is written, and inserts a `before insert row` record into `tg_log`.

```sql
CREATE OR REPLACE TRIGGER tg_bi_r
BEFORE INSERT ON t1
FOR EACH ROW
BEGIN
    :NEW.val := 'R_' || :NEW.val;
    INSERT INTO tg_log (seq, msg)
    SELECT NVL(MAX(seq), 0) + 1, 'before insert row' FROM tg_log;
END;
/
```

**tg_ai_r**: row-level `AFTER INSERT`, inserts an `after insert row` record into `tg_log` after each row is written to `t1`.

```sql
CREATE OR REPLACE TRIGGER tg_ai_r
AFTER INSERT ON t1
FOR EACH ROW
BEGIN
    INSERT INTO tg_log (seq, msg)
    SELECT NVL(MAX(seq), 0) + 1, 'after insert row' FROM tg_log;
END;
/
```

**tg_ai_s**: statement-level `AFTER INSERT`, inserts a single `after insert` record into `tg_log` after the entire `INSERT` statement completes.

```sql
CREATE OR REPLACE TRIGGER tg_ai_s
AFTER INSERT ON t1
BEGIN
    INSERT INTO tg_log (seq, msg)
    SELECT NVL(MAX(seq), 0) + 1, 'after insert' FROM tg_log;
END;
/
```

- Executing DML and viewing the results

  ```sql
  -- Clear the log for easier observation (optional).
  DELETE FROM tg_log;
  DELETE FROM t1;
  -- Insert a row.
  INSERT INTO t1 (id, val) VALUES (1, 'a');
  ```

  ```
  -- SELECT result:
  SELECT * FROM t1 ORDER BY id;

  ID                                       VAL       
  ---------------------------------------- ----------
  1                                        R_a       

  1 rows fetched.

  SELECT seq, msg FROM tg_log ORDER BY seq;

  SEQ                                      MSG                             
  ---------------------------------------- --------------------------------
  1                                        before insert                   
  2                                        before insert row               
  3                                        after insert row                
  4                                        after insert                    

  4 rows fetched.
  ```

- Disabling triggers (optional)

  ```sql
  ALTER TRIGGER tg_bi_r DISABLE;
  DELETE FROM t1;
  DELETE FROM tg_log;
  INSERT INTO t1 (id, val) VALUES (3, 'c');
  ALTER TRIGGER tg_bi_r ENABLE;
  ```

  ```
  -- SELECT result:
  SELECT * FROM t1;

  ID                                       VAL       
  ---------------------------------------- ----------
  3                                        c         

  1 rows fetched.


  SELECT seq, msg FROM tg_log ORDER BY seq;

  SEQ                                      MSG                             
  ---------------------------------------- --------------------------------
  1                                        before insert                   
  2                                        after insert row                
  3                                        after insert                    

  3 rows fetched.
  ```

- Dropping triggers and tables

  ```sql
  DROP TRIGGER IF EXISTS tg_bi_s;
  DROP TRIGGER IF EXISTS tg_bi_r;
  DROP TRIGGER IF EXISTS tg_ai_r;
  DROP TRIGGER IF EXISTS tg_ai_s;

  DROP TABLE IF EXISTS tg_log;
  DROP TABLE IF EXISTS t1;
  ```
