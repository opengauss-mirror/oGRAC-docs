# CREATE SEQUENCE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:04.977Z pushedAt=2026-08-17T01:38:47.624Z -->

## Function Description

A sequence is a special database object used to generate unique, ascending or descending numeric sequences. `CREATE SEQUENCE` adds a new sequence to the current database. The owner of the sequence is the user who created it.

## Precautions

- After a sequence is created, sequence values can be obtained using NEXTVAL and CURRVAL.

- Sequence names must be unique within the same schema.

- The `CACHE` option can be used to pre-allocate sequence values in memory to improve performance, but some sequence values may be lost in the event of an abnormal exit.

## Syntax

```sql
CREATE SEQUENCE [schema.]sequence_name
[INCREMENT BY INCREMENT_VALUE] | 
[ START WITH START_VALUE ] | 
[{MINVALUE MIN_VALUE} | {NOMINVALUE}]
[{MAXVALUE MAX_VALUE} | {NOMAXVALUE}]
[{CYCLE} | {NOCYCLE}]
[{CACHE CACHE_SIZE} | {NOCACHE}]
[{ORDER} | {NOORDER}];
```

## Parameter Description

- **[schema.]sequence_name**: Specifies the name of the sequence to be created. If a schema name is specified, the sequence will be created in that schema; otherwise, it will be created in the current schema.

- **INCREMENT BY INCREMENT_VALUE**: Specifies the step size of the sequence. A positive value generates an ascending sequence, and a negative value generates a descending sequence. The default value is 1.

- **START WITH START_VALUE**: Specifies the start value of the sequence. Default value: `MIN_VALUE` for an ascending sequence, and `MAX_VALUE` for a descending sequence.

- **[{MINVALUE MIN_VALUE} | {NOMINVALUE}]**: Specifies the minimum value of the sequence.

    - If MINVALUE is not specified or NOMINVALUE is specified, the default value for an ascending sequence is 1, and the default value for a descending sequence is -2^63+1.

- **[{MAXVALUE MAX_VALUE} | {NOMAXVALUE}]**: Specifies the maximum value of the sequence.

    - If MAXVALUE is not specified or NOMAXVALUE is specified, the default value for an ascending sequence is 2^63-1, and the default value for a descending sequence is -1.

- **[{CYCLE} | {NOCYCLE}]**: Allows the sequence to continue cycling after reaching `MAX_VALUE` or `MIN_VALUE`.

    - If NOCYCLE is specified, any call to NEXTVAL returns an error after the sequence reaches its maximum value.

    - If CYCLE is specified, the uniqueness of the sequence is not guaranteed.

    - The default value is NOCYCLE.

- **[{CACHE CACHE_SIZE} | {NOCACHE}]**: Specifies the number of sequence values pre-stored in memory for fast access.

    - If CACHE is specified, CACHE_SIZE sequence values are cached in memory. Each call to the nextval() function returns a sequence value from the cache. If the cache is empty, a sequence value is read from the database.

    - If CACHE is specified, the default value of CACHE_SIZE is 20, and the minimum value is 2.

    - If NOCACHE is specified, the sequence value is read from the database each time the nextval() function is called.

- **[{ORDER} | {NOORDER}]**: Specifies whether the sequence is generated in order.

    - If ORDER is specified, the sequence values are guaranteed to be generated in the order of requests.

    - If NOORDER is specified, the sequence is not generated in order.

    - The default value is NOORDER.

## Examples

```
-- Basic sequence creation example
-- Sequence name MY_SEQUENCE MIN_VALUE 1 MAX_VALUE 2^63-1 INCREMENT_BY 1 CYCLE_FLAG 0 ORDER_FLAG 0 CACHE_SIZE 20
SQL> CREATE SEQUENCE my_sequence;

Succeed.

SQL> SELECT * FROM ADM_SEQUENCES where SEQUENCE_NAME = 'MY_SEQUENCE';

SEQUENCE_OWNER                                                   SEQUENCE_NAME                                                    MIN_VALUE            MAX_VALUE            INCREMENT_BY         CYCLE_FLAG   ORDER_FLAG   CACHE_SIZE           LAST_NUMBER         
---------------------------------------------------------------- ---------------------------------------------------------------- -------------------- -------------------- -------------------- ------------ ------------ -------------------- --------------------
SYS                                                              MY_SEQUENCE                                                      1                    9223372036854775807  1                    0            0            20                   1                   

1 rows fetched.

-- Ascending sequence creation. Sequence name MY_SEQUENCE
-- Minimum value 10, maximum value 1000
-- Increment by 5, CYCLE TRUE, ORDER TRUE
-- Cache 50
SQL> CREATE SEQUENCE my_sequence
  2 INCREMENT BY 5
  3 START WITH 100
  4 MINVALUE 10
  5 MAXVALUE 1000
  6 CYCLE
  7 CACHE 50
  8 ORDER;

Succeed.

SQL> SELECT * FROM ADM_SEQUENCES where SEQUENCE_NAME = 'MY_SEQUENCE';

SEQUENCE_OWNER                                                   SEQUENCE_NAME                                                    MIN_VALUE            MAX_VALUE            INCREMENT_BY         CYCLE_FLAG   ORDER_FLAG   CACHE_SIZE           LAST_NUMBER         
---------------------------------------------------------------- ---------------------------------------------------------------- -------------------- -------------------- -------------------- ------------ ------------ -------------------- --------------------
SYS                                                              MY_SEQUENCE                                                      10                   1000                 5                    1            1            50                   100                 

1 rows fetched.

-- Descending sequence creation. Sequence name MY_SEQUENCE
-- Minimum value -2^63+1, maximum value 10000
-- Decrement by -10, CYCLE FALSE, ORDER FALSE
-- Cache 50
SQL> CREATE SEQUENCE my_sequence
  2 INCREMENT BY -10
  3 START WITH 10000
  4 NOMINVALUE
  5 MAXVALUE 10000
  6 NOCYCLE
  7 CACHE 50
  8 NOORDER;

Succeed.

SQL> SELECT * FROM ADM_SEQUENCES where SEQUENCE_NAME = 'MY_SEQUENCE';

SEQUENCE_OWNER                                                   SEQUENCE_NAME                                                    MIN_VALUE            MAX_VALUE            INCREMENT_BY         CYCLE_FLAG   ORDER_FLAG   CACHE_SIZE           LAST_NUMBER         
---------------------------------------------------------------- ---------------------------------------------------------------- -------------------- -------------------- -------------------- ------------ ------------ -------------------- --------------------
SYS                                                              MY_SEQUENCE                                                      -9223372036854775808 10000                -10                  0            0            50                   10000               

1 rows fetched.

```
