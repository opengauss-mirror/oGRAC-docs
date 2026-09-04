# DROP SEQUENCE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:58.346Z pushedAt=2026-08-17T07:54:18.809Z -->

## Function Description

Deletes a sequence from the database.

## Precautions

- Before dropping a sequence, ensure that no dependent objects (such as table defaults, triggers, etc.) are still referencing the sequence.

- The drop operation is irreversible and must be performed with caution.

## Syntax

```sql
DROP SEQUENCE [IF EXISTS] [Schema.]sequence_name;
```

## Parameter Description

- **IF EXISTS**: Optional. No error is reported when the sequence does not exist.

- **Schema.**: Optional. Specifies the schema name to which the sequence belongs.

- **sequence_name**: The name of the sequence to be dropped.

## Examples

```
-- Delete the sequence MY_SEQUENCE.
SQL> DROP SEQUENCE my_sequence;

Succeed.

SQL> DROP SEQUENCE IF EXISTS my_sequence;

Succeed.
```
