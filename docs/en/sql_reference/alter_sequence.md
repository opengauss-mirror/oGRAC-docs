# ALTER SEQUENCE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:04:05.047Z pushedAt=2026-08-27T08:13:14.993Z -->

## Function Description

Modifies the parameters of an existing sequence.

## Precautions

- When modifying certain attributes of a sequence (such as the minimum and maximum values), ensure that the newly set values are logically reasonable. For example, the minimum value should be less than the maximum value.

- If a sequence has reached its maximum value with `CYCLE` enabled, it will restart from the minimum value on the next value retrieval. If `CYCLE` is not enabled and the maximum is reached, an error will be raised upon the next value retrieval.

## Syntax

- Modify sequence attributes.

    ```
    ALTER SEQUENCE [schema.]sequence_name [ INCREMENT BY increment ]
        [ MINVALUE minvalue | NOMINVALUE ] [ MAXVALUE maxvalue | NOMAXVALUE ]
        [ CACHE cachevalue | NOCACHE ] [ CYCLE | NOCYCLE ];
    ```

## Parameter Description

- `schema`

    Specifies the username. It defaults to the current user.

- `sequence_name`

    Specifies the name of the sequence to be modified.

- `INCREMENT`

    Specifies the step size of the sequence. If the step size is set to a value greater than 0, the sequence ascends; if the step size is smaller than 0, the sequence descends. The default value is `1`.

- `MINVALUE minvalue | NOMINVALUE`

    Specifies the minimum value of the sequence. If `minvalue` is not declared or `NOMINVALUE` is declared, the default value is `1` for an ascending sequence or –2<sup>63</sup> + 1 for a descending sequence.

- `MAXVALUE maxvalue | NOMAXVALUE`

    Specifies the maximum value of the sequence. If `maxvalue` is not declared or `NOMAXVALUE` is declared, the default value is `-1` for a descending sequence and 2<sup>63</sup> – 1 for an ascending sequence.

- `CACHE cachevalue | NOCACHE`

    Specifies how many sequence numbers are to be pre-allocated and stored in memory for fast access. If it is not specified, the default value is `NOCACHE`, that is, the default number of sequence numbers is 1.

- `CYCLE | NOCYCLE`

    Allows the sequence to wrap around when `maxvalue` or `minvalue` has been reached.

    If `NOCYCLE` is specified, any calls to `nextval` after the sequence has reached its maximum value will return an error.

    If `CYCLE` is specified, sequence uniqueness cannot be guaranteed.

    The default value is `NOCYCLE`.

## Examples

```
-- Create an ascending sequence named serial, starting from 101.
SQL> CREATE SEQUENCE serial START WITH 101;

-- Modify the sequence step to 2.
SQL> ALTER SEQUENCE serial INCREMENT BY 2;

-- Modify the minimum value of the sequence to 90.
SQL> ALTER SEQUENCE serial MINVALUE 90;

-- Modify the maximum value of the sequence to 200.
SQL> ALTER SEQUENCE serial MAXVALUE 200;

-- Modify the cache value of the sequence to 10.
SQL> ALTER SEQUENCE serial CACHE 10;

-- Modify the sequence to CYCLE.
SQL> ALTER SEQUENCE serial CYCLE;

-- Delete the sequence.
SQL> DROP SEQUENCE serial;
```