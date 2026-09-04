# ALTER INDEX

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:04:05.172Z pushedAt=2026-08-27T08:05:35.452Z -->

## Function Description

Modifies the definition of an existing index.

## Precautions

Only the index owner or users granted the `ALTER ANY INDEX` permission can execute this command. Common users cannot modify system user objects.

## Syntax

- Rename a table index.

    ```
    ALTER INDEX [ schema_name.]index_name ON [ schema_name.]table_name
        RENAME TO [ schema_name.]new_name;
    ```

- Rebuild a table index or index partition.

    ```
    ALTER INDEX [ schema_name.]index_name ON [ schema_name.]table_name
        REBUILD [ PARTITION part_name[,...] | SUBPARTITION subpart_name[,...] ] 
        [ PARALLEL n | PCTFREE n | TABLESPACE tablespace_name ] [,...] ;
    ```

- Reclaim empty index pages.

    ```
    ALTER INDEX [ schema_name.]index_name ON [ schema_name.]table_name
        COALESCE
    ```

- Invalidate an index.

    ```
    ALTER INDEX [ schema_name.]index_name ON [ schema_name.]table_name
        UNUSABLE
    ```

- Modify the number of transaction slots on the initial data page of an index.

    ```
    ALTER INDEX [ schema_name.]index_name ON [ schema_name.]table_name
        INITRANS integer
    ```

- Modify a specified partition or subpartition of an index.

    ```
    ALTER INDEX [ schema_name.]index_name ON [ schema_name.]table_name
        MODIFY [ PARTITION part_name { COALESCE | UNUSABLE | INITRANS integer }
                 | SUBPARTITION subpart_name { COALESCE | UNUSABLE } ]
    ```

## Parameter Description

- `schema_name`: Specifies the schema name.

- `index_name`: Specifies the name of the index to be modified.

- `table_name`: Specifies the name of the table where the index is located.

- `new_name`: Specifies the new index name.

- `part_name`: Specifies the partition name.

- `subpart_name`: Specifies the subpartition name.

- `PARALLEL n`: Specifies the degree of parallelism when an index is rebuilt.

    - The value range of `n` is [1, 64].

    - Function-based indexes cannot be rebuilt in parallel.

- `PCTFREE n`: Specifies the percentage of free space reserved in each index data block. If a block's available space falls below this percentage, only updates to data in this block are permitted, and insertions are not allowed.

    - The value range of `n` is [0, 80], and the default value is `8`.

- `tablespace_name`: Specifies the tablespace name. An index is allowed to be rebuilt into a different tablespace.

- `INITRANS n`: Modifies the number of transaction slots on the initial data page of an index.

    - The value range of `n` is [1, 255].

    - The new value is valid only for newly allocated pages, and is invalid for old pages that have been allocated.

    - For a partitioned index, the `INITRANS` attribute of the index partition and subpartition is modified synchronously.

## Examples

```
SQL> CREATE TABLE alter_index_test(id INT) PARTITION BY RANGE(id)
    (PARTITION p1 VALUES LESS THAN (100), PARTITION p2 VALUES LESS THAN (200));

-- Create a partitioned index.
SQL> CREATE INDEX idx ON alter_index_test(id) LOCAL;

-- Rename the index.
SQL> ALTER INDEX idx ON alter_index_test RENAME TO new_idx;

-- Invalidate the index.
SQL> ALTER INDEX new_idx ON alter_index_test UNUSABLE;

-- Rebuild the index.
SQL> ALTER INDEX new_idx ON alter_index_test REBUILD PCTFREE 50;

-- Rebuild an index partition with a specified degree of parallelism.
SQL> ALTER INDEX new_idx ON alter_index_test REBUILD PARTITION p2 PARALLEL 2;

-- Specify INITRANS for partition p1.
SQL> ALTER INDEX new_idx ON alter_index_test MODIFY PARTITION p1 INITRANS 200;

SQL> DROP TABLE alter_index_test;
```