# DROP DIRECTORY

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:48.181Z pushedAt=2026-08-17T07:09:40.909Z -->

## Function Description

Deletes a directory object.

## Precautions

- Deleting a directory object requires the DROP ANY DIRECTORY permission.

## Syntax

**stmt:**

```sql
DROP DIRECTORY directory_name
```

## Parameter Description

- **directory_name**: Name of the directory object to be deleted

## Example

```
DROP DIRECTORY data_dir_0;

```
