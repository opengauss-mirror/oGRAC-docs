# Date Types

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:45.391Z pushedAt=2026-08-17T07:07:48.351Z -->

**Table 1** Date types

| Name | Description | Storage Space | Value Range |
| :------------ | :------------ | :------------ | :------------ |
| DATETIME/DATE | Stores date type data without time zone.<br>1. Stores year, month, day, hour, minute, and second. | 8 bytes | [0001-01-01 00:00:00, 9999-12-31 23:59:59] |
| TIMESTAMP[(n)] | Stores timestamp type data without time zone.<br>1. Stores year, month, day, hour, minute, second, and microsecond.<br>2. The valid range for `n` is [0,6], indicating the precision of the fractional seconds. `TIMESTAMP(n)` can also be written without the parameter, i.e., as `TIMESTAMP`, in which case the precision of the fractional seconds defaults to 6. | 8 bytes | [0001-01-01 00:00:00.000000, 9999-12-31 23:59:59.999999] |
| TIMESTAMP[(n)] WITH TIME ZONE | Stores timestamp type data with time zone.<br>1. Stores year, month, day, hour, minute, second, microsecond, and time zone.<br>2. The valid range for `n` is [0,6], indicating the precision of the fractional seconds. `TIMESTAMP(n)` can also be written without the parameter, i.e., as `TIMESTAMP`, in which case the precision of the fractional seconds defaults to 6. | 12 bytes | [0001-01-01 00:00:00.000000, 9999-12-31 23:59:59.999999] |
| TIMESTAMP[(n)] WITH LOCAL TIME ZONE | Stores timestamp type data with time zone. The time zone is not stored; data is converted to the database time zone's `TIMESTAMP` when stored, and converted to the current session's time zone `TIMESTAMP` when viewed by users.<br>1. Stores year, month, day, hour, minute, second, and microsecond.<br>2. The valid range for `n` is [0,6], indicating the precision of the fractional seconds. `TIMESTAMP(n)` can also be written without the parameter, i.e., as `TIMESTAMP`, in which case the precision of the fractional seconds defaults to 6. | 8 bytes | [0001-01-01 00:00:00.000000, 9999-12-31 23:59:59.999999] |

**Table 2** shows the template that can be used to format date and time values.

**Table 2** Patterns for date/time formatting

<table>
<tr>
<td>Category</td>
<td>Pattern</td>
<td>Description</td>
</tr>
<tr>
<td rowspan="4">Hour</td>
<td>HH</td>
<td>Hour of the day (01-12)</td>
</tr>
<tr>
<td>HH12</td>
<td>Hour of the day (01-12)</td>
</tr>
<tr>
<td>HH24</td>
<td>Hour of the day (00-23)</td>
</tr>
<tr>
<td>TZH</td>
<td>Time zone hour</td>
</tr>
<tr>
<td rowspan="2">Minute</td>
<td>MI</td>
<td>Minute (00-59)</td>
</tr>
<tr>
<td>TZM</td>
<td>Time zone minute</td>
</tr>
<tr>
<td rowspan="5">Second</td>
<td>FF</td>
<td>Microsecond (000000-999999)</td>
</tr>
<tr>
<td>FF3</td>
<td>Microsecond (000-999)</td>
</tr>
<tr>
<td>FF6</td>
<td>Microsecond (000000-999999)</td>
</tr>
<tr>
<td>SS</td>
<td>Second (00-59)</td>
</tr>
<tr>
<td>SSSSS</td>
<td>Seconds past midnight (0-86399)</td>
</tr>
<tr>
<td rowspan="2">A.M./P.M.</td>
<td>AM or A.M.</td>
<td>Morning indicator</td>
</tr>
<tr>
<td>PM or P.M.</td>
<td>Afternoon indicator</td>
</tr>
<tr>
<td rowspan="4">Year</td>
<td>YYYY</td>
<td>Year (4 digits)</td>
</tr>
<tr>
<td>YYY</td>
<td>Last three digits of the year</td>
</tr>
<tr>
<td>YY</td>
<td>Last two digits of the year</td>
</tr>
<tr>
<td>Y</td>
<td>Last digit of the year</td>
</tr>
<tr>
<td rowspan="3">Month</td>
<td>MONTH</td>
<td>Full uppercase month name</td>
</tr>
<tr>
<td>MON</td>
<td>Abbreviated uppercase month name</td>
</tr>
<tr>
<td>MM</td>
<td>Month number (01-12)</td>
</tr>
<tr>
<td rowspan="5">Day</td>
<td>DAY</td>
<td>Full uppercase day name</td>
</tr>
<tr>
<td>DY</td>
<td>Abbreviated uppercase day name</td>
</tr>
<tr>
<td>DDD</td>
<td>Day of the year (001-366)</td>
</tr>
<tr>
<td>DD</td>
<td>Day of the month (01-31)</td>
</tr>
<tr>
<td>D</td>
<td>Day of the week (1-7; 1 for Sunday)</td>
</tr>
<tr>
<td rowspan="2">Week</td>
<td>W</td>
<td>Week of the month (1–5, week 1 starts on the first day of the month)</td>
</tr>
<tr>
<td>WW</td>
<td>Week of the year (1–53, week 1 starts on the first day of the year)</td>
</tr>
<tr>
<td rowspan="1">Century</td>
<td>CC</td>
<td>Century (2 digits)</td>
</tr>
<tr>
<td rowspan="1">Quarter</td>
<td>Q</td>
<td>Quarter of the year</td>
</tr>
</table>

Example:

```
SQL> select to_char(systimestamp, 'YYYY-MM-DD HH:MI:SS A.M.');

TO_CHAR(SYSTIMESTAMP, 'YYYY-MM-DD HH:MI:SS A.M.')
-------------------------------------------------
2025-11-12 09:37:31 AM

1 rows fetched.

SQL> select to_char(systimestamp, 'YYYY-MM-DD HH:MI:SSXFF4');

TO_CHAR(SYSTIMESTAMP, 'YYYY-MM-DD HH:MI:SSXFF4')
------------------------------------------------
2025-11-12 09:27:23.6485

1 rows fetched.

-- Create a table.
SQL> CREATE TABLE date_t1 (a datetime,b timestamp(6), c timestamp(4) with time zone, d TIMESTAMP(2) WITH LOCAL TIME ZONE); 

-- Insert data.
SQL> INSERT INTO date_t1 VALUES ('2025-11-12 09:37:31', systimestamp, systimestamp, systimestamp);

-- View the data.
SQL> SELECT * FROM date_t1;

A                      B                                C                                        D
---------------------- -------------------------------- ---------------------------------------- --------------------------------
2025-11-12 09:37:31    2025-11-20 17:35:31.740367       2025-11-20 17:35:31.7404 +08:00          2025-11-20 17:35:31.74

1 rows fetched.

SQL> DROP TABLE date_t1;
```
