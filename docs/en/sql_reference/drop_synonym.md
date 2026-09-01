# DROP SYNONYM

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:16.755Z pushedAt=2026-08-17T08:05:35.582Z -->

## Function Description

Deletes one or more synonyms from the database. A synonym is an alias for a database object, and dropping a synonym does not affect the original object it points to.

## Precautions

Exercise caution when dropping public synonyms or synonyms with dependent objects. It is recommended to use `IF EXISTS` to avoid errors when the object does not exist.

## Syntax

### Dropping a Private Synonym

```sql
DROP SYNONYM [IF EXISTS] synonym_name [FORCE];
```

### Dropping a Public Synonym

```
DROP PUBLIC SYNONYM [IF EXISTS] synonym_name [FORCE];
```

## Parameter Description

| Name | Description |
|------|------|
| `IF EXISTS` | Optional. Does not raise an error but returns a notice if the specified synonym does not exist. |
| `synonym_name` | Required. The name of the synonym to be dropped. |
| `FORCE` | Optional. Forcibly drops the synonym even if it has dependent objects. |
| `PUBLIC` | Optional. Specifies that the synonym to be dropped is a public synonym. |

## Examples

### Dropping a Private Synonym

```
-- Drop synonym emp_syn.
DROP SYNONYM emp_syn;

```

### Dropping a Public Synonym

```
-- Drop public synonym public_emp.
DROP PUBLIC SYNONYM public_emp;
```

### Using the IF EXISTS Option

```
-- If the synonym exists, drop it; if not, do not raise an error.
DROP SYNONYM IF EXISTS non_existent_syn;
```

### Using the FORCE Option

```
-- Forcibly drop the synonym even if it has dependent objects.
DROP SYNONYM emp_syn FORCE;

-- Forcibly drop the public synonym.
DROP PUBLIC SYNONYM public_emp FORCE;
```
