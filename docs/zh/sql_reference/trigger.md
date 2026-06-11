# TRIGGER

## 功能描述

触发器是与表或视图关联的特殊存储过程，当对其关联的表或视图执行特定的 DML（INSERT、UPDATE、DELETE）操作时自动触发执行。oGRAC 触发器支持 BEFORE、AFTER、INSTEAD OF 三种触发时机，以及语句级和行级两种触发级别。

## 注意事项

- 执行创建、删除触发器需要拥有 **CREATE TRIGGER** 系统权限；在其他用户 schema 的对象上创建还需相应对象权限。
- 不能在 **SYS** 所属对象上创建触发器。
- 单表触发器数量上限为 **8** 个。
- “OF col” 列数据类型不能为 LOB。
- 不能在本地临时表（名称以 `#` 开头）上创建触发器。
- 触发器体内对正在被DML修改的基表执行部分SQL，可能触发变异表错误（`ERR_TAB_MUTATING`）。
- 触发器内部不能出现 DDL 和 DCL 语句，普通用户不能创建系统用户对象。
- 每个触发器对象只能绑定一种时机（语句级或行级之一），不支持复合触发器（`COMPOUND TRIGGER`）。
- 创建触发器时，最后的”/”符号为必须添加的结束符，否则会报错。

---

## 语法格式

### 创建触发器

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

参数说明：
- **CREATE [OR REPLACE] TRIGGER [IF NOT EXISTS] [schema.]trigger_name**

  创建或替换触发器。`OR REPLACE` 表示若同名触发器已存在则覆盖；`IF NOT EXISTS` 表示若已存在则不报错、不创建。

- **[schema.]**

  可选。触发器所属 schema；省略时使用当前会话 schema。

- **trigger_name**

  触发器名称，须符合对象命名规则。

- **{ BEFORE | AFTER | INSTEAD OF }**

  触发时机，三选一，紧跟在触发器名之后。

  - **BEFORE**：在 DML 真正修改数据之前执行。
  - **AFTER**：在 DML 修改数据之后执行。
  - **INSTEAD OF**：替代对视图的原 DML 执行，仅允许创建在视图上；不能用于普通表。

- **{ INSERT | DELETE | UPDATE [OF col1 [, col2 ...]] }**

  触发事件，指定在何种 DML 下激活触发器。

  - **INSERT**：插入行时触发。
  - **DELETE**：删除行时触发。
  - **UPDATE**：更新行时触发。
  - **UPDATE OF** ***col1*** **[,** ***col2*** **...]**

    仅当SET子句中修改了所列列时才触发；列必须存在于基表、可见，且不能为LOB类型。

- **[ OR { INSERT | DELETE | UPDATE [OF ...] } ... ]**

  可选。用 `OR` 连接多个事件，使同一触发器响应多种 DML，例如 `INSERT OR UPDATE ON tbl`。

- **ON [schema.]table_or_view**

  触发器绑定的表或视图。

  - 普通表、无日志表、事务级/会话级临时表：仅支持 **BEFORE** / **AFTER**。
  - 视图：仅支持 **INSTEAD OF**。
  - 本地临时表（`#` 开头）：不支持创建触发器。

- **[ FOR EACH ROW ]**

  可选。指定为**行级**触发器；省略则为**语句级**触发器。

  - 行级：每影响一行执行一次触发器体；可使用`:NEW`、`:OLD`。
  - 语句级：每条DML语句只执行一次；不能使用`:NEW`、`:OLD`。
  - `BEFORE` / `AFTER`的行级触发器必须显式写出`FOR EACH ROW`。
  - `INSTEAD OF`触发器通常为行级，写法上同样使用`FOR EACH ROW`。

- **[ DECLARE ... ]**

  可选。触发器局部变量、游标等声明区，语法与 PL/SQL 块相同。

- **BEGIN ... END**

  触发器执行体，PL/SQL语句序列。语句须以`/`结束提交（与SQL*Plus习惯一致）。

示例：

- 建表。

    ```sql
    DROP TABLE IF EXISTS test;
    CREATE TABLE test (
        emp_id   NUMBER PRIMARY KEY,
        emp_name VARCHAR2(50),
        job      VARCHAR2(50),
        salary   NUMBER(10, 2)
    );
    ```

- **test_trigger_br**：行级BEFORE INSERT，每插入一行前将`emp_name`、`job`转为大写。

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

- **test_trigger_bs**：语句级 BEFORE INSERT，每条 INSERT 开始前对局部变量赋值。

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

- **tri_sal_audit**：行级 AFTER UPDATE OF salary，仅当更新 `salary` 列时在行写入后执行（体为空占位）。

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

- 建视图 **v_emp**，再创建 **tri_v_emp_io**：INSTEAD OF INSERT，将对视图的插入转为对基表 `test` 的插入。

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

---

## 删除触发器

```sql
DROP TRIGGER [IF EXISTS] [schema.]trigger_name;
```

参数说明：  
- **DROP TRIGGER [IF EXISTS] [schema.]trigger_name**

  删除指定触发器。

- **[IF EXISTS]**

  可选。触发器不存在时不报错。

- **[schema.]**

  可选。触发器所有者；省略时表示当前 schema。

- ***trigger_name***

  要删除的触发器名。

示例：

- 删除触发器。

    ```sql
    DROP TRIGGER test_trigger_br;
    ```

- 触发器不存在时不报错。

    ```sql
    DROP TRIGGER IF EXISTS test_trigger_br;
    ```

---

## 修改触发器

```sql
ALTER TRIGGER [schema.]trigger_name ENABLE;
ALTER TRIGGER [schema.]trigger_name DISABLE;
```

参数说明：      
- **ENABLE**

  启用已禁用的触发器，之后 DML 将按类型与时机再次执行该触发器。

- **DISABLE**

  禁用触发器，之后 DML 跳过该触发器，不执行其触发器体。

- **[schema.]**

  可选。触发器所有者。

- ***trigger_name***

  要启用或禁用的触发器名。

示例：

- 前置：建表并创建 **test_trigger_br**（与上文「创建触发器」示例一致）。

    ```sql
    DROP TABLE IF EXISTS test;
    CREATE TABLE test (
        emp_id   NUMBER PRIMARY KEY,
        emp_name VARCHAR2(50),
        job      VARCHAR2(50),
        salary   NUMBER(10, 2)
    );

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

- 禁用触发器（之后对该表的 DML 不再执行该触发器）。

    ```sql
    ALTER TRIGGER test_trigger_br DISABLE;
    ```

- 重新启用触发器。

    ```sql
    ALTER TRIGGER test_trigger_br ENABLE;
    ```

---

## 行级伪列 :NEW / :OLD

行级触发器或 INSTEAD OF 触发器中，通过伪列访问当前行的新旧值（非表的真实列）。

### 语法格式

访问操作表列：

```sql
{ :OLD | :NEW }.col
```

参数说明：

- **col**：

  列名，和数据库大小写敏感性保持一致。

访问伪列：

```sql
{ :OLD | :NEW }.{ rowid | rowscn }
```

### 访问规则：
- **:NEW.column_name**

  新行各列的值。

  - 可用于：**BEFORE INSERT**、**BEFORE UPDATE**、**INSTEAD OF** 行级触发器（只读或赋值）。
  - 在 **BEFORE INSERT** / **BEFORE UPDATE** 中对 `:NEW.col` 赋值可改变即将写入的数据。
  - 在 **AFTER** 触发器中一般只读，不能修改 `:NEW`。
  - 支持带引号列名，如 `:NEW."ColName"`。

- **:OLD.column_name**

  旧行各列的值。

  - 可用于：**BEFORE/AFTER UPDATE**、**BEFORE/AFTER DELETE**、**INSTEAD OF** 行级触发器。
  - **不允许**对 `:OLD.col` 赋值。

- **:NEW.rowid** / **:OLD.rowid**

  当前行的 ROWID，类型为字符串。

- **:NEW.rowscn** / **:OLD.rowscn**

  当前行的 SCN 相关信息，类型为大整数。

**限制：**

- 伪列只能出现在行级或 INSTEAD OF 触发器中；语句级触发器使用会报错。
- `:NEW` 的修改仅允许在 **BEFORE INSERT**、**BEFORE UPDATE** 行级触发器中。

示例:

- 触发器表列使用示例（`:OLD.col`、`:NEW.col`）：

    ```sql
    DROP TRIGGER IF EXISTS TEST_TRIGGER_COL;
    DROP TABLE IF EXISTS TEST_TRIGGER;

    CREATE TABLE TEST_TRIGGER(A INT, B INT);
    INSERT INTO TEST_TRIGGER VALUES(1, 2);

    --创建触发器TEST_TRIGGER_COL（行级 AFTER UPDATE，打印表列:OLD.A与:NEW.A）。
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

    -- 开启服务端输出（否则 DBE_OUTPUT.PRINT_LINE 不会显示在客户端）。
    SET SERVEROUTPUT ON;

    -- 更新表TEST_TRIGGER，触发器打印更新前后A列取值（只执行下面一条 UPDATE，分号结束）。
    UPDATE TEST_TRIGGER SET A = 10 WHERE A = 1;
    ```

    UPDATE后显示结果：


    ```sql
    :OLD.A = 1
    :NEW.A = 10

    1 rows affected.
    ```

    若出现 `0 rows affected`，说明没有 `A = 1` 的行（请先确认 `INSERT` 已执行且数据仍在表中）。

- 触发器伪列使用示例（`:OLD.rowid`、`:OLD.rowscn`、`:NEW.rowid`、`:NEW.rowscn`）：

    ```sql
    DROP TRIGGER IF EXISTS TEST_TRIGGER_PSEUDO;
    DROP TABLE IF EXISTS TEST_TRIGGER;

    CREATE TABLE TEST_TRIGGER(A INT, B INT);
    INSERT INTO TEST_TRIGGER VALUES(1, 2);

    --创建触发器TEST_TRIGGER_PSEUDO（行级 AFTER UPDATE，打印rowid/rowscn伪列）。
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

    -- 开启服务端输出
    SET SERVEROUTPUT ON;

    -- 更新表TEST_TRIGGER，触发器打印更新前后rowid/rowscn。
    UPDATE TEST_TRIGGER SET A = 10 WHERE A = 1;
    ```

    UPDATE后显示结果：

    ```sql
    :OLD.rowid000000000271400000
    :OLD.rowscn8923265734799361
    :NEW.rowid000000000271400000
    :NEW.rowscn8923265734799361

    1 rows affected.
    ```

---

## 完整流程示例（基于表的 DML 触发器）

以下示例用表 `t1`（待插入数据）、表 `tg_log`（记录触发器执行轨迹），在 **INSERT** 场景下演示四种时机的触发顺序，并完成触发器的创建、验证与删除。各触发器向 `tg_log` 插入一条说明文字（如 `before insert`），便于对照执行顺序。

- 准备表

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

- 创建触发器


  **tg_bi_s**：语句级 BEFORE INSERT，整条 INSERT 开始前向 `tg_log` 插入一条 `before insert` 记录。

    ```sql
    CREATE OR REPLACE TRIGGER tg_bi_s
    BEFORE INSERT ON t1
    BEGIN
        INSERT INTO tg_log (seq, msg)
        SELECT NVL(MAX(seq), 0) + 1, 'before insert' FROM tg_log;
    END;
    /
    ```

  **tg_bi_r**：行级 BEFORE INSERT，每行写入前给 `val` 加 `R_` 前缀，并向 `tg_log` 插入 `before insert row` 记录。

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

  **tg_ai_r**：行级 AFTER INSERT，每行写入 `t1` 后向 `tg_log` 插入 `after insert row` 记录。

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

  **tg_ai_s**：语句级 AFTER INSERT，整条 INSERT 结束后向 `tg_log` 插入一条 `after insert` 记录。

    ```sql
    CREATE OR REPLACE TRIGGER tg_ai_s
    AFTER INSERT ON t1
    BEGIN
        INSERT INTO tg_log (seq, msg)
        SELECT NVL(MAX(seq), 0) + 1, 'after insert' FROM tg_log;
    END;
    /
    ```

- 执行 DML 并查看结果

  ```sql
  -- 清空日志便于观察（可选）
  DELETE FROM tg_log;
  DELETE FROM t1;
  -- 插入一条数据
  INSERT INTO t1 (id, val) VALUES (1, 'a');
  ```


  ```sql
  -- SELECT结果：
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

- 禁用触发器（可选）

  ```sql
  ALTER TRIGGER tg_bi_r DISABLE;
  DELETE FROM t1;
  DELETE FROM tg_log;
  INSERT INTO t1 (id, val) VALUES (3, 'c');
  ALTER TRIGGER tg_bi_r ENABLE;
  ```

  ```sql
  -- SELECT 结果：
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

- 删除触发器与表

  ```sql
  DROP TRIGGER IF EXISTS tg_bi_s;
  DROP TRIGGER IF EXISTS tg_bi_r;
  DROP TRIGGER IF EXISTS tg_ai_r;
  DROP TRIGGER IF EXISTS tg_ai_s;

  DROP TABLE IF EXISTS tg_log;
  DROP TABLE IF EXISTS t1;
  ```