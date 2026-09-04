# Interval Types

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:36.775Z pushedAt=2026-08-17T09:37:30.586Z -->

**Table 1** Interval types

| Name  | Description  | Storage Size  | Range  |
| :------------ | :------------ | :------------ | :------------ |
| INTERVAL YEAR[(n)] TO MONTH  | Stores an interval of years and months.<br>1. The valid range for `n` is [0,4], indicating the precision of years. The default value is 2.  |  4 bytes   |  [-9999-11, +9999-11]  |
| INTERVAL DAY[(n1)] TO SECOND[(n2)]  | Stores an interval of days (including day, hour, minute, second, and microsecond).<br>1. The valid range for `n1` is [0,7], indicating the precision of days. The default value is 2.<br>2. The valid range for `n2` is [0,6], indicating the precision of fractional seconds. It defaults to 6 if not specified.  |  8 bytes   |  [-9999999 23:59:59.999999, +9999999 23:59:59.999999]  |

Examples

```
-- Create a table.
SQL> CREATE TABLE day_type_tab (a int,b INTERVAL DAY(3) TO SECOND (4), c INTERVAL YEAR(4) TO MONTH); 

-- Insert data.
SQL> INSERT INTO day_type_tab VALUES (1, INTERVAL '3' DAY, INTERVAL '4' MONTH);

-- View data.
SQL> SELECT * FROM day_type_tab;

A                                        B                        C
---------------------------------------- ------------------------ ----------
1                                        +003 00:00:00.0000       +0000-04

1 rows fetched.

--The right operand is a string. Before comparison, the system converts the string to the INTERVAL DAY TO SECOND type.
SQL>  SELECT * FROM day_type_tab where b = '003 00:00:00.0000';

A                                        B
---------------------------------------- ------------------------
1                                        +003 00:00:00.0000

1 rows fetched.

SQL> SELECT * FROM day_type_tab WHERE b = '002 00:00:00.0000';

A                                        B
---------------------------------------- ------------------------

0 rows fetched.

-- Drop a table.
SQL> DROP TABLE day_type_tab;
```
