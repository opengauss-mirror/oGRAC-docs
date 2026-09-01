# Numeric Types

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:45.741Z pushedAt=2026-08-17T10:25:18.302Z -->

**Table 1** Integer types

| Name | Description | Storage Space | Range |
| :------------ | :------------ | :------------ | :------------ |
| BINARY_INTEGER | Stores a 32-bit signed integer. | 4 bytes | -2^31 to 2^31 - 1 |
| BINARY_UINT32 | Stores a 32-bit unsigned integer. | 4 bytes | 0 to 2^32 - 1 |
| BINARY_BIGINT | Stores a 64-bit signed integer. | 8 bytes | -2^63 to 2^63 - 1 |

Examples

```
-- Create a table with BINARY_INTEGER, BINARY_UINT32, and BINARY_BIGINT data types.
SQL> CREATE TABLE int_type_t2 
(
    a BINARY_INTEGER, 
    b BINARY_UINT32,
    c BINARY_BIGINT
);

-- Insert data.
SQL> INSERT INTO int_type_t2 VALUES(100, 10, 1000);

-- View data.
SQL> SELECT * FROM int_type_t2;

A            B                                        C
------------ ---------------------------------------- --------------------
100          10                                       1000

1 rows fetched.

-- Drop the table.
SQL> DROP TABLE int_type_t2;
```

**Table 2** Floating-point type

| Name  | Description  | Storage Space  | Range  |
| :------------ | :------------ | :------------ | :------------ |
| BINARY_DOUBLE  | Stores a 64-bit double-precision floating-point number.  |  8 bytes   |  [-1.79E+308, +1.79E+308]   |

> **NOTE**
>
> - This data type is imprecise by nature. Some values cannot be precisely converted to the internal format and are stored as approximations. Consequently, there may be some discrepancies when the stored data is printed out. If you intend to use an imprecise type for any critical or complex calculations, especially those that heavily depend on boundary conditions (infinity/underflow), you should carefully evaluate the SQL logic and app implementation. Directly comparing two floating-point values may not always yield the expected result.
> - The data type supports numeric literals suffixed with f or d (case-insensitive). Both represent 8-byte floating-point numbers, with the same value range as BINARY_DOUBLE. For regular SQL statements, the ability to accept floating-point literals ending with f or d is governed by the GUC parameter use_bison_parser. This parameter is set to on by default. If it is not enabled, you must manually set it to true.

Examples

```
-- Create a table with floating-point data.
SQL> CREATE TABLE float_type_t2 
(
    a BINARY_DOUBLE
);

-- Insert data.
SQL> INSERT INTO float_type_t2 VALUES(1234567.89);

SQL> alter system set use_bison_parser = false;

SQL> INSERT INTO float_type_t2 VALUES(1234567.89F);

OG-00636, [1:34]Invalid number 

SQL> alter system set use_bison_parser = true;

SQL> INSERT INTO float_type_t2 VALUES(1234567.89F);
SQL> INSERT INTO float_type_t2 VALUES(1234567.89f);
SQL> INSERT INTO float_type_t2 VALUES(1234567.89D);
SQL> INSERT INTO float_type_t2 VALUES(1234567.89d);

-- View data.
SQL> SELECT * FROM float_type_t2;

A
--------------------
1234567.89
1234567.89
1234567.89
1234567.89
1234567.89

5 rows fetched.

-- Drop the table.
SQL> DROP TABLE float_type_t2;
```

**Table 3** High-precision numeric types

| Name  | Description  | Storage Space  | Range  |
| :------------ | :------------ | :------------ | :------------ |
| NUMBER/DECIMAL/NUMERIC[(p[,s])]  | Stores a high-precision floating-point number.<br>1. The valid range for `p` is [1,38], indicating the maximum precision that can be stored.<br>2. The valid range for `s` is [-84,127], indicating the number of significant digits after the decimal point. If `s` is negative, the corresponding number of digits to the left of the decimal point are rounded off.<br>3. If neither `p` nor `s` is specified, no restriction is imposed on the digits after the decimal point, and a maximum of 40 significant digits can be stored.<br>4. If `p` is specified but `s` is not specified or `s=0`, the NUMBER type has no fractional part.  |  4 to 24 bytes   |  (-1.0E128, 1.0E128)   |

Examples

```
-- Create a table with data of different precision and decimal place types.
SQL> CREATE TABLE number_type_t2 
(
    a NUMBER, 
    b NUMBER(12),
    c NUMBER(12, 3),
    d NUMBER(12, -2)
);

-- Insert data.
SQL> INSERT INTO number_type_t2 VALUES(1234567.89, 1234567.89, 1234567.89, 1234567.89);

-- View data.
SQL> SELECT * FROM number_type_t2;

A                                        B                                        C                                        D                                  
---------------------------------------- ---------------------------------------- ---------------------------------------- ----------------------------------------
1234567.89                               1234568                                  1234567.89                               1234600                            

1 rows fetched.

-- Set parameters.
SQL> alter system set use_bison_parser = true;

-- Insert data.
SQL> INSERT INTO number_type_t2 VALUES(1234567.89f, 1234567.89F, 1234567.89d, 1234567.89D);

-- View data.
SQL> SELECT * FROM number_type_t2;

A                                        B                                        C                                        D                                       
---------------------------------------- ---------------------------------------- ---------------------------------------- ----------------------------------------
1234567.89                               1234568                                  1234567.89                               1234600                                 
1234567.89                               1234568                                  1234567.89                               1234600                                 

2 rows fetched.

-- Drop the table.
SQL> DROP TABLE number_type_t2;
```

> **NOTE**
>
> - Table 4 lists the data types to which the data type keywords are actually mapped under the control of the USE_NATIVE_DATATYPE parameter. This parameter is of Boolean type and is used to resolve compatibility between different databases by distinguishing numeric type keywords. The default value is TRUE.
>
> - Some keywords are not controlled by the USE_NATIVE_DATATYPE parameter, including BINARY_INTEGER, BINARY_BIGINT, and BINARY_DOUBLE.

**Table 4** Data type keyword mapping

| Keyword | Mapped Data Type (USE_NATIVE_DATATYPE=FALSE) | Mapped Data Type (USE_NATIVE_DATATYPE=TRUE) |
| :------------ | :------------ | :------------ |
| BIGINT | NUMBER(38) | BINARY_BIGINT |
| DOUBLE | NUMBER | BINARY_DOUBLE |
| FLOAT | NUMBER | BINARY_DOUBLE |
| INT/INTEGER | NUMBER(38) | BINARY_INTEGER |
| UINT/INTEGER UNSIGNED | NUMBER(38) | BINARY_UINT32 |
| REAL | NUMBER | BINARY_DOUBLE |
| SMALLINT | NUMBER(38) | BINARY_INTEGER |
| TINYINT | NUMBER(38) | BINARY_INTEGER |
