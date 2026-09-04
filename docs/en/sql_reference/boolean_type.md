# Boolean Type

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:32.528Z pushedAt=2026-08-27T10:51:15.855Z -->

**Table 1** Boolean type

| Name | Description | Storage Space | Range |
| :------------ | :------------ | :------------ | :------------ |
| BOOLEAN/BOOL | Stores Boolean data | 4 bytes | TRUE, FALSE |

- Valid literal values for the "true" state include:

    `TRUE`, `true`, `'T'`, `'t'`, `'TRUE'`, `'true'`, `'1'`, and all non-zero integers.

- Valid literal values for the "false" state include:

    `FALSE`, `false`, `'F'`, `'f'`, `'FALSE'`, `'false'`, `'0'`, and `0`.

Example:

```
-- Create a table.
SQL> CREATE TABLE bool_type_t1  
(
    BT_COL1 BOOLEAN,
    BT_COL2 TEXT
);

-- Insert data.
SQL> INSERT INTO bool_type_t1 VALUES (TRUE, 'sic est');

SQL> INSERT INTO bool_type_t1 VALUES (FALSE, 'non est');

-- View the data.
SQL> SELECT * FROM bool_type_t1;

BT_COL1 BT_COL2
------- ----------------------------------------------------------------
TRUE    sic est
FALSE   non est

2 rows fetched.

SQL> SELECT * FROM bool_type_t1 WHERE bt_col1 = 't';

BT_COL1 BT_COL2
------- ----------------------------------------------------------------
TRUE    sic est

1 rows fetched.

-- Delete the table.
SQL> DROP TABLE bool_type_t1;

```