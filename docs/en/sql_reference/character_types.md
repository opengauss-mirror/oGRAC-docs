# Character Types

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:51.202Z pushedAt=2026-08-25T02:39:20.234Z -->

**Table 1**  Character types

| Name | Description | Storage Space |
| :------------ | :------------ | :------------ |
| CHAR(*size* [BYTE \| CHAR]) | Stores fixed-length strings. The size can be specified in bytes or characters.<br>1. <code>BYTE</code> (default) measures the size in bytes.<br>2. <code>CHAR</code> measures the size in characters. | 1–8000 bytes |
| NCHAR(*size*) | Equivalent to CHAR(*size* CHAR) for storing fixed-length strings. | 1–8000 bytes |
| NATIONAL CHARACTER(*size*) | Equivalent to CHAR(*size* CHAR) for storing fixed-length strings, provided that <code>use_bison_parser</code> is set to <code>true</code>. | 1–8000 bytes |
| NATIONAL CHAR(*size*) | Equivalent to CHAR(*size* CHAR) for storing fixed-length strings, provided that <code>use_bison_parser</code> is set to <code>true</code>. | 1–8000 bytes |
| CLOB/NCLOB/TEXT/LONGTEXT/LONG | Stores large object variable-length strings. | 0 to (4 GB – 1) bytes |
| VARCHAR/VARCHAR2(*size* [BYTE \| CHAR]) | Stores variable-length strings.<br>1. <code>size</code> specifies the maximum number of bytes or characters that can be stored.<br>2. <code>BYTE</code> (default) measures the size in bytes.<br>3. <code>CHAR</code> measures the size in characters. | 1–8000 bytes |
| NVARCHAR/NVARCHAR2(*size*) | Equivalent to VARCHAR(*size* CHAR) for storing variable-length strings. | 1–8000 bytes |
| NATIONAL CHARACTER VARYING(*size*) | Equivalent to VARCHAR(*size* CHAR) for storing variable-length strings, provided that <code>use_bison_parser</code> is set to <code>true</code>. | 1–8000 bytes |
| NATIONAL CHAR VARYING(*size*) | Equivalent to VARCHAR(*size* CHAR) for storing variable-length strings, provided that <code>use_bison_parser</code> is set to <code>true</code>. | 1–8000 bytes |
| NCHAR VARYING(*size*) | Equivalent to VARCHAR(*size* CHAR) for storing variable-length strings, provided that <code>use_bison_parser</code> is set to <code>true</code>. | 1–8000 bytes |
| ROWID | Approximately equivalent to CHAR(18) for storing strings in a specific format, which consists of only letters, digits, slashes (/), and plus signs (+). | 18 bytes |

> **NOTE:**
>
>- Currently, UTF-8 and GBK character sets are supported. In the UTF-8 character set, Chinese characters and full-width characters occupy 2 to 6 bytes, while digits and English characters each occupy one byte. In the GBK character set, Chinese characters and full-width characters each occupy 2 bytes, while digits and English characters each occupy one byte.
>- The use of N-prefixed literals like `select N'1234'` is supported, provided that the `use_bison_parser` parameter is set to `true`.

- ROWID

    A valid ROWID data type consists of 18 characters. It can be split into four segments, with lengths of 6, 3, 6, and 3 characters from left to right, whose abbreviated names are `object`, `rfile`, `block`, and `row`, respectively.
    Each character can be considered as a base-64 digit, where:
    `A` to `Z` represents the numbers 0 to 25, `a` to `z` represents 26 to 51, `0` to `9` represents 52 to 61, `+` represents 62, and `/` represents 63.
    For example, given the ROWID `AAAAA/AA/AAAAA/IAA`, `object` is `AAAAA/`, `rfile` is `AA/`, `block` is `AAAAA/`, and `row` is `IAA`.
    Here, `object`, `rfile`, and `block` each represent the number 63, and `row` represents 2<sup>15</sup> (001000 000000 000000).

    After obtaining the values of each ROWID segment, you need to check whether the values fall within the valid ranges. Specifically:
    0 <= `object` < 2<sup>32</sup>, 0 <= `rfile` < 2<sup>10</sup>, 0 <= `block` < 2<sup>22</sup>, and 0 <= `row` < 2<sup>15</sup>.
    In the example above, `row` is 2<sup>15</sup> (`I` represents the digit 8, and `IAA` is 8 × 2<sup>12</sup>), which cannot be represented with 15 bits (`row` >= 2<sup>15</sup>). This means it is an invalid `row` segment, and therefore an invalid ROWID format.

    As an exception, a ROWID value consisting of 18 `A` characters is invalid.

Example:

```
-- Create a table with fixed-length and variable-length character type data.
SQL> CREATE TABLE char_type_t1 
(
    a CHAR(5),
    b VARCHAR(5),
    c NCHAR(10),
    d CLOB,
    e NVARCHAR2(10)
);

-- Insert data.
SQL> INSERT INTO char_type_t1 VALUES ('ok', 'ok', 'abcdef', 'abcdef', 'abcdef');

SQL> INSERT INTO char_type_t1 VALUES ('good', 'good', 'good', 'good', 'good');

SQL> SELECT char_length(a), char_length(b), char_length(c), char_length(d), char_length(e) from char_type_t1;

CHAR_LENGTH(A)       CHAR_LENGTH(B)       CHAR_LENGTH(C)       CHAR_LENGTH(D)       CHAR_LENGTH(F)
-------------------- -------------------- -------------------- -------------------- --------------------
5                    2                    10                   6                    6
5                    4                    10                   4                    4

2 rows fetched.

-- An error is reported when the length of the inserted data exceeds the defined length of its type.
SQL> INSERT INTO char_type_t1(a, b) VALUES ('too long', 'too long');

CT-00698, The size(8) of value can't larger than defined size(5) of char
-- Delete the table.
SQL> DROP TABLE char_type_t1;

SQL> CREATE TABLE test_clob(c1 clob, c2 nclob, c3 text, c4 longtext, c5 long);

SQL> INSERT INTO test_clob values('abcdefg', 'abcdefg', 'abcdefg', 'abcdefg', 'abcdefg');

SQL> SELECT * FROM test_clob;

C1                                                               C2                                                               C3                                                               C4                                                               C5                                                              
---------------------------------------------------------------- ---------------------------------------------------------------- ---------------------------------------------------------------- ---------------------------------------------------------------- ----------------------------------------------------------------
abcdefg                                                          abcdefg                                                          abcdefg                                                          abcdefg                                                          abcdefg                                                         

1 rows fetched.

SQL> SHOW CREATE TABLE test_clob;

CREATE TABLE "TEST_CLOB"
(
  "C1" CLOB,
  "C2" CLOB,
  "C3" CLOB,
  "C4" CLOB,
  "C5" CLOB
)
TABLESPACE "SYSTEM"
INITRANS 2
MAXTRANS 255
PCTFREE 8
FORMAT ASF;

SQL> ALTER SYSTEM SET use_bison_parser = true;

SQL> CREATE TABLE test_char1
(
    c1 national character varying(10),
    c2 national char varying(10),
    c3 nchar varying(10)
);

SQL> SHOW CREATE TABLE test_char1;

CREATE TABLE "TEST_CHAR1"
(
    "C1" VARCHAR(10 CHAR),
    "C2" VARCHAR(10 CHAR),
    "C3" VARCHAR(10 CHAR)
)
TABLESPACE "SYSTEM"
INITRANS 0
MAXTRANS 255
PCTFREE 0
FORMAT ASF;

SQL> CREATE TABLE test_char2
(
    c1 national character,
    c2 national char,
    c3 nchar
);

SQL> SHOW CREATE TABLE test_char2;

CREATE TABLE "TEST_CHAR2"
(
    "C1" CHAR(1 CHAR),
    "C2" CHAR(1 CHAR),
    "C3" CHAR(1 CHAR)
)
TABLESPACE "SYSTEM"
INITRANS 0
MAXTRANS 255
PCTFREE 0
FORMAT ASF;

SQL> CREATE TABLE test_char3(
    c1 national character(10),
    c2 national char(10),
    c3 nchar(10)
);

SQL> SHOW CREATE TABLE test_char3;

CREATE TABLE "TEST_CHAR3"
(
    "C1" CHAR(10 CHAR),
    "C2" CHAR(10 CHAR),
    "C3" CHAR(10 CHAR)
)
TABLESPACE "SYSTEM"
INITRANS 0
MAXTRANS 255
PCTFREE 0
FORMAT ASF;

SQL> SELECT n'abcd';

N'ABCD'
-------
abcd   

1 rows fetched.

SQL> SELECT N'abcd';

N'ABCD'
-------
abcd   

1 rows fetched.

SQL> SELECT n'1234';

N'1234'
-------
1234   

1 rows fetched.

SQL> SELECT N'1234';

N'1234'
-------
1234   

1 rows fetched.

SQL> ALTER SYSTEM SET use_bison_parser = false;
```