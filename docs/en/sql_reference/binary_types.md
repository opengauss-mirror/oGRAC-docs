# Binary Types

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:34.726Z pushedAt=2026-08-27T10:49:33.765Z -->

**Table 1** Binary types

| Name | Description | Storage Space |
| :------------ | :------------ | :------------ |
| BINARY(*size*) | Stores fixed-length binary data.<br>If the input length is less than *size*, the input is right padded with 0. | 1–8000 bytes |
| VARBINARY(*size*) | Stores variable-length binary data. | 1–8000 bytes |
| IMAGE | Stores large object data. It is the large object type of VARBINARY. | 0 to (4 GB – 1) bytes |
| RAW(*size*) | Stores variable-length binary data. | 1–8000 bytes |
| BLOB/BYTEA | Stores variable-length binary large object data. It is the large object type of RAW. The input is a hexadecimal string. | 0 to (4 GB – 1) bytes |

Example:

```
-- Create a table.
SQL> CREATE TABLE blob_type_t1 
(
    a INTEGER,
    b BLOB,
    c RAW,
    d BYTEA,
    e binary(10),
    f varbinary(10),
    g image
) ;

-- Insert data.
SQL> INSERT INTO blob_type_t1 VALUES(10,empty_blob(),
HEXTORAW('DE'),'\xDEADBEEF', '\xDEADBEEF', '\xDEADBEEF', '\xDEADBEEF');

-- Query data in the table.
SQL>  SELECT * FROM blob_type_t1;

A                                        B                                                                C                                                                D                                                                E                                                                F                                                                G
---------------------------------------- ---------------------------------------------------------------- ---------------------------------------------------------------- ---------------------------------------------------------------- ---------------------------------------------------------------- ---------------------------------------------------------------- ----------------------------------------------------------------
10                                                                                                        DE                                                               DEADBEEF                                                         \xDEADBEEF                                                       \xDEADBEEF                                                       \xDEADBEEF

1 rows fetched.

-- Delete the table.
SQL> DROP TABLE blob_type_t1;

```