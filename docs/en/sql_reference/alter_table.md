# ALTER TABLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:49.796Z pushedAt=2026-08-27T11:17:34.162Z -->

## Function Description

Modifies the structure of a database table, covering changes to columns and constraints. The detailed functions are as follows:

- Adding, deleting, and renaming data columns, as well as modifying their attributes

- Adding and removing constraints

- Enabling or disabling existing constraints

- Renaming data tables

- Splitting an existing partition into smaller units

- Exchanging data between two tables or partitions

## Precautions

- Only users with the `ALTER TABLE` or `ALTER ANY TABLE` system permission can perform this operation. Non-privileged users cannot modify objects owned by the `SYS` account.

- If the table name, column name, or constraint name specified in the command conflicts or is invalid, or if the data in the table does not conform to the constraint being enabled in a `NOVALIDATE` state, the system returns an explicit error message.

- To modify an attribute of a column in a table, the value of that column must be `NULL` in all existing records. If the column contains non-`NULL` values and is not a partition key, only the following changes are allowed: BINARY, INT, CHAR, and VARCHAR types can have their length or size increased; and high-precision numeric types can have their range extended, provided that both the new number of decimal digits and the new number of integer digits are not less than their original values. Other data types and operations are not allowed.

- When adding a new column or modifying an existing column, you can specify `PRIMARY KEY` or `UNIQUE` constraints in the column definition. However, inline declaration of `FOREIGN KEY` constraints is not supported.

- The `ALTER TABLE` command does not apply to external tables.

- This operation cannot be performed during a database restart or transaction rollback.

## Syntax

```sql
ALTER TABLE [ schema_name. ]table_name
{ alter_table_properties
| column_clauses
| partition_clauses
| set_interval_clause
| logic_replication_clauses
}

```

- `alter_table_properties` syntax components:

    ```
    { physical_attributes_clause
    | RENAME TO new_table_name
    | AUTO_INCREMENT [ = ] value
    }
    ```

    - `physical_attributes_clause` syntax components:

        ```
        { PCTFREE integer
        | INITRANS integer
        | APPENDONLY { ON | OFF }
        | storage_alter_clause
        }
        ```

        - `storage_alter_clause` syntax components:

            ```
            STORAGE ( MAXSIZE { UNLIMITED | integer [K | M | G | T] } )
            ```

- `column_clauses` syntax components:

    ```
    { add_column_clause
    | modify_column_clause
    | drop_column_clause
    | rename_column_clause
    }
    ```

    - `add_column_clause` syntax components:

        ```
        -- Add a single data column.
        ADD [ COLUMN ] column_name datatype_name [ DEFAULT expr [ON UPDATE expr ] ]
        [ COMMENT 'string' ] [ COLLATE collation_name ] [AUTO_INCREMENT] [ inline_constraint ]
        -- Add multiple data columns.
        ADD ( [ COLUMN ] { column_name datatype_name [ DEFAULT expr [ON UPDATE expr ] ]
        [ COMMENT 'string' ] [ COLLATE collation_name ] [AUTO_INCREMENT] [ inline_constraint ] }
        [ , ... ] )
        ```

- `inline_constraint` syntax components:

            ```
            [ CONSTRAINT constraint_name ]{ [ NOT ] NULL
            | CHECK( expr )
            | PRIMARY KEY
            | UNIQUE
            }[ ... ]
            ```

- `modify_column_clause` syntax components:

        ```
        -- Modify the column definition.
        MODIFY ( { column_name [ new_datatype_name ] [ DEFAULT expr [ ON UPDATE expr ] ]
        [ COMMENT string ]
        [ COLLATE collation_name ]
        [ inline_constraint ] } [ , ... ]
        )
        -- Reclaim the space occupied by an LOB column.
        MODIFY LOB(column_name) (SHRINK SPACE)
        ```

    - `drop_column_clause` syntax component:

        ```
        DROP [ COLUMN ] column_name
        ```

    - `rename_column_clause` syntax component:

        ```
        RENAME COLUMN old_name TO new_name
        ```

- `partition_clauses` syntax components:

    ```
    { add_partition_clause
    | drop_partition_clause
    | truncate_partition_clause
    | coalesce_partition_clause
    | split_partition_clause
    | modify_partition_clause
    }
    ```

    - `add_partition_clause` syntax components:

        ```
        ADD PARTITION partition_name
        { VALUES LESS THAN ( { partition_value | MAXVALUE } [ , ... ] )
        | VALUES ( partition_value [ , ... ] | DEFAULT )
        }
        [ TABLESPACE tablespace_name ]
        [ PCTFREE integer ][ storage_clause ]
        { FORMAT CSF | [ COMPRESS ]
        [ ( { SUBPARTITION subpartition_name (
        { VALUES LESS THAN ( { subpartition_value | MAXVALUE } [, ... ] )
        | VALUES ( { subpartition_value [, ... ] | DEFAULT } [, ... ] )
        }
        [ TABLESPACE tablespace_name ]
        ) } [, ... ] ) ]
        }

        MODIFY PARTITION partition_name ADD SUBPARTITION
        { VALUES LESS THAN ( { subpartition_value | MAXVALUE } [, ... ] )
        | VALUES ( subpartition_values [, ... ] | DEFAULT )
        }
        [ TABLESPACE tablespace_name ]
        ```

        - `storage_clause` syntax components:

            ```
            STORAGE ( { INITIAL integer [K | M | G | T]
            |MAXSIZE { UNLIMITED | integer [K | M | G | T] }
            } [ ...] )
            ```

    - `drop_partition_clause` syntax components:

        ```
        DROP { PARTITION partition_name | SUBPARTITION subpartition_name }
        ```

    - `truncate_partition_clause` syntax components:

        ```
        TRUNCATE { PARTITION partition_name | SUBPARTITION subpartition_name }
        [ DROP STORAGE | REUSE STORAGE | PURGE ]
        ```

    - `coalesce_partition_clause` syntax components:

        ```
        COALESCE PARTITION
        MODIFY PARTITION partition_name COALESCE SUBPARTITION
        ```

- `split_partition_clause` syntax components:

    ```
    SPLIT PARTITION partition_name
    AT (range_value)
    INTO
    (
        PARTITION part_name1 [ TABLESPACE space_name ],
        PARTITION part_name2 [ TABLESPACE space_name ]
    )
    [ UPDATE GLOBAL INDEXES ]

    SPLIT SUBPARTITION subpartition_name
    AT (range_value)
    INTO
    (
        SUBPARTITION subpart_name1 [ TABLESPACE space_name ],
        SUBPARTITION subpart_name2 [ TABLESPACE space_name ]
    )
    [ UPDATE GLOBAL INDEXES ]
    ```

- `modify_partition_clause` syntax components:

    ```
    MODIFY PARTITION partition_name { INITRANS integer | storage_alter_clause }
    ```

    - `storage_alter_clause` syntax components:

      ```
      STORAGE (MAXSIZE { UNLIMITED | integer [K | M | G | T] } )
      ```

- `set_interval_clause` syntax component:

    ```
    SET INTERVAL([interval_value])
    ```

- `logic_replication_clauses` syntax components:

    ```
    [([ partition_name | subpartition_name ][ , ... ])] ADD LOGICAL LOG(UNIQUE index_name)|
    [([ partition_name | subpartition_name ][ , ... ])] ADD LOGICAL LOG(PRIMARY KEY) | DROP LOGICAL LOG
    ```

## Parameter Description

- `[schema_name.]`: Indicates the schema name. If it is not explicitly specified, the system uses the name of the currently logged-in user as the schema name by default.

- `table_name`: Indicates the name of the target table to be modified. The table must already exist in the database.

- `alter_table_properties`: Modifies the physical storage properties of a data table. For example, `LOB_storage_clause` is used to specify that large object (LOB) columns are stored in independent segments, either inline or out-of-line. Currently, the storage engine supports only out-of-line storage.

    - `physical_attributes_clause`:

        - `INITRANS integer`: Adjusts the number of pre-allocated transaction slots on each initial data page in a data table. The value range of this parameter is [1, 255].

            > **NOTE:**
            >
            > - This modification takes effect only on newly allocated data pages; existing allocated pages are not affected.
            > - For partitioned tables, this operation updates the `INITRANS` attribute of the table partitions and all secondary partitions simultaneously.

        - `storage_alter_clause`: Sets the maximum storage space that a data table can use.

            - `UNLIMITED`: Indicates that no upper limit is set for the storage space of this table.

            - `integer [K | M | G | T]`: Explicitly sets the maximum storage space for the table. The allowed range is [1 MB, 1 TB].

    - `APPENDONLY { ON | OFF }`: Specifies the concurrent insertion policy. When it is set to `ON`, space is increased independently for each insertion thread, which can improve insertion performance under high concurrency. The default value is `OFF`. Exercise caution when enabling this function, as improper use may lead to reduced storage space utilization.

        - `ON`: Enables independent space expansion.

            > **Important Notes:**
            >
            > - For partitioned tables, when `APPENDONLY ON` is set, special attention is required in parallel insertion scenarios: You must pre-plan data to ensure that the data inserted by each parallel thread does not span partitions (achieving a "one thread per partition" mapping).
            > - You are not advised to set the `APPENDONLY ON` option for hash partitioned tables.

        - `OFF`: Disables independent space expansion and executes regular concurrent insertion.

    - `PCTFREE integer`: Defines the percentage of free space reserved in a data block. When the available space in a data block falls below this percentage, the block only allows update operations and prohibits new data insertion. The value range is [0, 80], and the default value is `8`.

    - `RENAME TO new_table_name`: Renames a data table.

    - `AUTO_INCREMENT [ = ] value`: Modifies the starting sequence value of the auto-increment column in the table. If not specified, it starts from `1` by default.

- `column_clauses`: Alters the table structure, including adding, deleting, and modifying data columns.

    - `add_column_clause`: Adds a new data column to the table.

    - `DEFAULT —— [ON UPDATE expr]`: Specifies an expression used to calculate the default value of a column. During DDL creation, if `DEFAULT` specifies a constant expression, the data type of this constant will be checked for data compatibility.

        - **`[ON UPDATE expr]`**: Specifies a default update expression to fill the column when a row is updated without an explicit value assignment (for syntax compatibility).


        - **In `INSERT` and `UPDATE` operations, the maximum length of the expression text following `DEFAULT` is limited to 1,024 English characters. If the length exceeds the maximum, the error message `GS-00611, default value string is too long, exceed 1024.` is displayed.**

    - `COMMENT 'string'`: Adds a comment to the column. Column comments can be viewed by querying the `MY_COL_COMMENTS` system view.

    - `COLLATE collation_name`: Defines a collation rule for the column data. The rule specifies how data is stored and compared.

        `collation_name` indicates the collation name. Its values can be:

        - `UTF8_BIN`: Applicable to the UTF-8 character set. Characters are considered as binary strings and are compared bit by bit from the most significant bit to the least significant bit. The characters to be compared are case-sensitive.

        - `UTF8_GENERAL_CI`: Applicable to the UTF-8 character set. The characters to be compared are case-insensitive.

        - `UTF8_UNICODE_CI`: Applicable to the UTF-8 character set. The characters to be compared are case-insensitive.

        - `GBK_BIN`: Applicable to the GBK character set. The characters to be compared are case-sensitive.

        - `GBK_CHINESE_CI`: Applicable to the GBK character set. The characters to be compared are case-insensitive.

    - `inline_constraint`: Indicates the inline column constraint, which is included in the column definition. Currently, `NULL`, `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `UNIQUE INDEX`, `FOREIGN KEY`, and `CHECK` constraints are supported.

    - `modify_column_clause`: Modifies the attributes of one or more columns, including changing the data type, adding compliant column constraints, and shrinking the space occupied by LOB columns. You cannot modify the attributes of columns that are used in a function-based index.

        To change the data type of a column into an incompatible one, ensure that the table is empty or all values in the column are `NULL`. To change the data type to a compatible type, ensure that the table contains data and not all values in the column to be modified are `NULL`. Currently, the following compatible data type conversions are supported:

        - Conversion between the `VARCHAR` type and the `CHAR` type (the length cannot be less than that before the conversion).

        - Length limit expansion for `VARCHAR`, `CHAR`, `BINARY`, and `INT` types.

        - Value range expansion for high-precision data types, such as `NUMBER` (the new number of decimal places (`scale`) and the new number of integer digits (`precision` – `scale`) must each be no less than their respective values before modification).

            - `new_datatype_name`: Specifies the target data type of the column after modification.

    - `drop_column_clause`: Drops a specified column from the table.

        - `DROP [ COLUMN ] column_name`: Drops a column. `column_name` is the name of the column to be dropped.

    - `rename_column_clause`: Renames an existing column in the table.

        - `RENAME COLUMN old_name TO new_name`: Renames a column.

            - `old_name`: Specifies the original name of the column to be renamed.

            - `new_name`: Specifies the new name of the column after renaming.

- `partition_clauses`: Includes partition-related operations.

    - `add_partition_clause`: Adds a new partition to a partitioned table.

        - `VALUES LESS THAN`: Defines the upper bound (exclusive) of a new partition for range partitioning.

        - `partition_value`: Specifies the limit of a partition.

        - `MAXVALUE`: Indicates the maximum possible value allowed for a partition, typically used for the last partition.

        - `VALUES`: Defines a specific set of values that a new partition contains in list partitioning.

        - `DEFAULT`: Creates a default partition that holds data not assigned to any other partitions in list partitioning.

        - `INITIAL integer [K | M | G | T]`: Specifies the initial storage size of a new partition. By default, a new partition is allocated with one extent. You can customize the initial size using this parameter. The value range is [64 KB, 1 TB].

        - `MAXSIZE { UNLIMITED | integer [K | M | G | T] }`: Specifies the maximum storage space that the partition can use.

            - `UNLIMITED`: Indicates that no upper limit is imposed on the storage space of the partition.

            - `integer [K | M | G | T]`: Explicitly sets the maximum storage space for the partition. The value range is [1 MB, 1 TB].

        - `FORMAT CSF`: Adding a hash partition with the CSF attribute will cause data redistribution. Therefore, an error may be reported when a hash partition with the CSF attribute is added. Whether the error is reported depends on whether the CSF attribute constraint is met.

        - `COMPRESS`: Adds a compressed partition. Ensure that the tablespace where the compressed partition is located has the compression attribute file. Otherwise, an error will be reported when data is inserted.

    - `drop_partition_clause`

        - `DROP PARTITION partition_name`

            Deletes a partition. `partition_name` specifies the partition name.

        - `DROP SUBPARTITION subpartition_name`

            Deletes a subpartition. `subpartition_name` specifies the name of the subpartition.

    - `split_partition_clause`

        Splits a (sub)partition into two. Data in the original partition is redistributed to the new partitions. Currently, only range partitions can be split.

        - `(sub)partition_name`

            Specifies the name of the (sub)partition to be split.

        - `(sub)part_name1 (sub)part_name2`

            Specifies the names of the new (sub)partitions after splitting. Note that the two names must be different. If a partition is split, only the last new partition can use the original partition name.

        - `range_value`

            Specifies the boundary of values for splitting.

        - `UPDATE GLOBAL INDEXES`

            - If `UPDATE GLOBAL INDEXES` is specified and the partitioned table has global indexes, the global indexes are automatically rebuilt after data redistribution is complete.

            - If it is not specified, the global indexes become invalid.

    - `exchange_partition_clause`

        Exchanges partitions.

        - `WITH TABLE`

            Specifies the name of the common table to be exchanged.

        - `INCLUDING | EXCLUDING INDEXES`

            - `INCLUDING INDEXES`

                Indicates that indexes need to be exchanged.

            - `EXCLUDING INDEXES`

                Indicates that indexes do not need to be exchanged.

        - `WITH | WITHOUT VALIDATION`

            - `WITH VALIDATION`

                Indicates that data needs to be verified.

            - `WITHOUT VALIDATION`

                Indicates that data does not need to be verified.

        - `partition_clause`

            Specifies the information about the partition to be exchanged.

            - `partition_name`

                Specifies the name of the partition to be exchanged.

            - `FOR (part_key value)`

                If it is difficult to obtain the partition name, you can set the partition key value to specify the partition to be exchanged.

        > **NOTE:** Constraints on partition exchange
        >
        > 1. RCR tables and indexes cannot be exchanged.
        > 2. Clauses involving `FOREIGN KEY` constraints, such as `CASCADE`, are not supported. If any of the two tables to be exchanged has a foreign key relationship, an error is reported during partition exchange.
        > 3. Exchange is not allowed if the compression and `NOLOGGING INSERT` attributes of the two partitions are different.
        > 4. Exchange is not allowed when the partitions to be exchanged contain auto-increment columns.
        > 6. Exchange is allowed only when the table definition, index definition, and column definition of the two partitions are the same.

    - `modify_partition_clause`

        Modifies the attributes of a partition.

        - `partition_name`

            Specifies the name of the partition to be modified.

        - `INITRANS integer`

            Specifies the number of transaction slots on the initial data page of the modified partition. The value range is [1, 255].

            - If the `INITRANS` attribute of a partition is modified, the `INITRANS` attribute of all its subpartitions is modified accordingly.

            - The new value takes effect for newly allocated pages, but is invalid for old pages that have been allocated.

            > **NOTE:** `INITRANS` usage
            >
            > 1. In high-concurrency OLTP systems, when frequent `UPDATE`/`INSERT` operations cause transaction slot contention, you can set `INITRANS` to 4–8 to reduce dynamic expansion overhead.
            > 2. The default `INITRANS` for index blocks is `2`, which may not meet the requirements of high-concurrency writes. You can set it to `3`.

        - `storage_alter_clause`

            Specifies the maximum table storage space.

            - `UNLIMITED`

                Indicates that the table storage space is unlimited.

            - `integer[K|M|G|T]`

                Sets the maximum table storage space. The value range is [1 MB, 1 TB].

    - `coalesce_partition_clause`

        - `COALESCE PARTITION`

            Inserts data from the last partition into an earlier partition, and then deletes the last partition.

            - The `COALESCE PARTITION` statement can be executed only for hash partitions. You do not need to specify the partition name in this statement.

            - If there is only one partition, the `COALESCE PARTITION` statement cannot be executed; otherwise, an error is reported.

    - `[PARTITION | SUBPARTITION] NOLOGGING`

        - `NOLOGGING`

        Enables or disables the `NOLOGGING INSERT` attribute for a partition.

        - `PARTITION NOLOGGING`

        Enables or disables the `NOLOGGING INSERT` attribute for a partition.

        - `SUBPARTITION NOLOGGING`

        Enables or disables the `NOLOGGING INSERT` attribute for a subpartition.

        The `NOLOGGING INSERT` attribute of a table is independent of that of its partitions. Whether a partition has the `NOLOGGING INSERT` attribute enabled is determined by itself, but not by the `NOLOGGING INSERT` attribute of the table. The `NOLOGGING INSERT` attribute of a subpartition is determined by that of its parent partition. If the `NOLOGGING INSERT` attribute is enabled for a partition, the attribute is enabled for all its subpartitions. However, setting the `NOLOGGING INSERT` attribute of a subpartition does not influence that of its parent partition.

        When using this feature, note the following constraints:

        - The `NOLOGGING INSERT` operation is performed to improve the performance of importing a large amount of data to the database. Do not perform the `NOLOGGING INSERT` operation on a table or partition at the same time as other normal services. Otherwise, errors may occur in normal services due to concurrent undo log reading because the `NOLOGGING INSERT` operation does not record undo logs.

        - After data is imported to the database using `NOLOGGING INSERT`, you are advised to commit transactions in a timely manner and manually trigger data flushing before starting other related services. This effectively ensures data security and prevents data from being re-imported to the database due to data loss caused by power failures.

        - `NOLOGGING INSERT` does not record undo data. After data is imported, it is strongly recommended that transactions be committed in a timely manner before other services are executed to avoid errors caused by the failure to access historical data. In addition, because the undo log is not generated, if the `NOLOGGING INSERT` operation is performed in a transaction, the rollback operation can be performed successfully, but no data modification is performed in the database. Therefore, the data of a transaction that has executed `NOLOGGING INSERT` cannot be rolled back to a historical version. Since no redo/undo log is generated, once any exception occurs, the database can no longer guarantee data consistency. In this case, you need to delete the table data and re-execute the import operation.

        - The session-level `NOLOGGING INSERT` is intended for internal tool use only, and cannot be used for customer services. Instead, the table partition-level `NOLOGGING INSERT` is recommended for customer services.

        - `NOLOGGING INSERT` is not allowed if logical replication is enabled or the primary/standby deployment is adopted. If `NOLOGGING INSERT` is enabled for any table or partition object in the database, standby nodes cannot be dynamically added.

        - Temporary tables do not support the `NOLOGGING INSERT` attribute. As they inherently have the attribute of not recording redo logs, setting `NOLOGGING` is unnecessary.

        - **If data already exists in the table or partition, enabling the `NOLOGGING INSERT` operation is not allowed. Notes on version compatibility**: A database version that supports the table partition-level `NOLOGGING INSERT` feature cannot be downgraded to a version that does not support the feature, but can be upgraded. If both target and source versions support the feature, the upgrade or downgrade can be performed normally (without considering the impact of other features). In addition, before the upgrade or downgrade, you need to manually check whether the database system contains `NOLOGGING` objects. If these objects do not require the `NOLOGGING` attribute, you are advised to disable the attribute before the upgrade or downgrade.

        - **If the `NOLOGGING INSERT` attribute of an object is not disabled before backup and restoration, the `NOLOGGING` attribute will still be valid in the new environment after restoration.** If it is unnecessary, disable the attribute of the object before the backup.

- `logic_replication_clauses`

  Enables or disables the table-level or partition-level logical replication function.

    - `(sub)partition_name`

      - You can add a table (sub)partition name in the parentheses following the table name to set partition-level logical replication. This operation supports multiple supplementary configurations for partitions that have not yet been enabled for logical replication. However, table-level logical replication and partition-level logical replication cannot be switched directly.

      - The `SYS` user does not load any data dictionary. Therefore, the `SYS` user cannot enable the partition-level logical replication function through multiple supplementary operations.

    - `ADD LOGICAL LOG(PRIMARY KEY)`

      Enables the table-level logical replication function based on the primary key.

    - `ADD LOGICAL LOG(UNIQUE index_name)`

      Enables the logical replication function based on the unique index.

    - `DROP LOGICAL LOG`

      Disables table-level and partition-level logical replication.

- `rename_column_clause`

  Renames a table. Only tables in your own schemas can be renamed. Tables in the system tablespace cannot be renamed.

- `set_interval_clause`

  Sets a partition interval, which is valid only for partitioned tables.

    - `SET INTERVAL()`: Changes an interval partitioned table to a range partitioned table.

    - `SET INTERVAL(interval_value)`: Changes the interval for an interval partitioned table. `interval_value` specifies the specific interval value.

## Examples

- Add a column.

```
-- Delete the table test.
DROP TABLE IF EXISTS test;
-- Create the table test.
CREATE TABLE test(student_id INT NOT NULL, course_name VARCHAR(30), course_start_date DATETIME, score INT);
-- Add the column full_score.
ALTER TABLE test ADD full_score INT;
```

- Modify the data type of a column.

```
ALTER TABLE test MODIFY course_name VARCHAR(20);
```

- Add a `PRIMARY KEY` constraint.

```
ALTER TABLE t_or2union_1 ADD CONSTRAINT pk_a PRIMARY KEY (a);
```

- Drop a column.

```
ALTER TABLE test DROP score;
```

- Rename a table.

```
ALTER TABLE test RENAME TO test2025;
```

- Create a partitioned table.

```
-- Delete the table test_partition.
DROP TABLE IF EXISTS test_partition;
-- Create the partitioned table test_partition.
CREATE TABLE test_partition(
  student_id INT NOT NULL,
  course_name CHAR(20),
  exam_date DATETIME,
  score INT)
PARTITION BY RANGE(student_id)
(
PARTITION test_partition1 VALUES LESS THAN(100),
PARTITION test_partition2 VALUES LESS THAN(200),
PARTITION test_partition3 VALUES LESS THAN(300),
PARTITION test_partition4 VALUES LESS THAN(400)
);
```

- Add partitions `test_partition5` and `test_partition6`.

```
ALTER TABLE test_partition ADD PARTITION test_partition5 VALUES LESS THAN(450);
ALTER TABLE test_partition ADD PARTITION test_partition6 VALUES LESS THAN(MAXVALUE);
```

- Delete partitions `test_partition3` and `test_partition4`.

```
-- Delete the partition test_partition3.
ALTER TABLE test_partition DROP PARTITION test_partition3;

-- Clear the partition test_partition4.
ALTER TABLE test_partition TRUNCATE PARTITION test_partition4;
```

- Split a partition.

```
ALTER TABLE test_partition SPLIT PARTITION test_partition5 AT(430) INTO (PARTITION p1, PARTITION p2);
```

- Change the `MAXSIZE` value for a table partition.

```
-- Delete the table.
DROP TABLE IF EXISTS test_partition;
-- Create a table.
CREATE TABLE test_partition(o_id INT, o_char VARCHAR2(900)) STORAGE (MAXSIZE 5M INITIAL 1M)
PARTITION BY RANGE(o_id)(
PARTITION p1 VALUES LESS THAN(20) STORAGE (MAXSIZE 3M INITIAL 1M),
PARTITION p2 VALUES LESS THAN(50)
);
-- Change the MAXSIZE value for the table storage space.
ALTER TABLE test_partition STORAGE (MAXSIZE 10M);
-- Add a partition. The initial size is 5 MB and the minimum size is 2 MB.
ALTER TABLE test_partition ADD PARTITION p3 VALUES LESS THAN(80) STORAGE (MAXSIZE 5M INITIAL 2M);
-- Change the maximum size of the p1 partition to 2 MB.
ALTER TABLE test_partition MODIFY PARTITION p1 STORAGE (MAXSIZE 2M);
```

- Change the `INITRANS` value for the table and table partition.

```
-- Delete the table.
DROP TABLE IF EXISTS test_partition;
-- Create a table.
CREATE TABLE test_partition(o_id INT, o_char VARCHAR2(1000)) INITRANS 10
PARTITION BY RANGE(o_id)(
PARTITION p1 VALUES LESS THAN(2) INITRANS 5,
PARTITION p2 VALUES LESS THAN(3)
);

-- Change the INITRANS value of the table.
ALTER TABLE test_partition INITRANS 20;
-- Change the INITRANS value of a table partition.
ALTER TABLE test_partition MODIFY PARTITION p1 INITRANS 10;
```

- Enable or disable table-level logical replication.

```
-- Delete the table.
DROP TABLE IF EXISTS test_partition;
-- Create a table.
CREATE TABLE test_partition(
  student_id INT PRIMARY KEY,
  course_name VARCHAR(50))
PARTITION BY RANGE(student_id)(
PARTITION p1 VALUES LESS THAN(10),
PARTITION p2 VALUES LESS THAN(50),
PARTITION p3 VALUES LESS THAN(100)
);
-- Enable table-level logical replication.
ALTER TABLE test_partition ADD LOGICAL LOG(PRIMARY KEY);
-- Disable table-level logical replication.
ALTER TABLE test_partition DROP LOGICAL LOG;
```

- Enable or disable partition-level logical replication.

```
-- Delete the table.
DROP TABLE IF EXISTS test_partition;
-- Create a table.
CREATE TABLE test_partition(
  student_id INT PRIMARY KEY,
  course_name VARCHAR(50))
PARTITION BY RANGE(student_id)(
PARTITION p1 VALUES LESS THAN(10),
PARTITION p2 VALUES LESS THAN(50),
PARTITION p3 VALUES LESS THAN(100)
);
-- Enable partition-level logical replication.
ALTER TABLE test_partition(p1,p2) ADD LOGICAL LOG(PRIMARY KEY);
-- Delete a partition. When a table partition is deleted, partition-level logical replication is disabled.
ALTER TABLE test_partition DROP PARTITION p1;
-- Disable table-level logical replication.
ALTER TABLE test_partition DROP LOGICAL LOG;
```