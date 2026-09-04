# CREATE SYNONYM

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:15:02.899Z -->

## Function Description

Creates an alias for a database object, simplifying the writing and use of SQL statements. A synonym can point to a database object such as a table, providing an abstraction layer to hide the real name and owner of an object.

## Precautions

To create a synonym, you must have the access permission on the target object. Public synonyms are visible to all users, so use them with caution to avoid naming conflicts.

## Syntax

### Creating a Private Synonym

```sql
CREATE [OR REPLACE] SYNONYM synonym_name FOR object_name;
```

### Creating a Public Synonym

```
CREATE [OR REPLACE] PUBLIC SYNONYM synonym_name FOR object_name;
```

## Parameter Description

| Name | Description |
|------|-------------|
| OR REPLACE | Optional. If the synonym already exists, replaces it instead of reporting an error. |
| PUBLIC | Optional. Creates a public synonym accessible to all users. If not specified, creates a private synonym visible only to the creator. |
| synonym_name | Name of the synonym to be created. |
| object_name | Name of the object that the synonym points to. |

## Examples

### Example 1: Creating a Private Synonym

```
CREATE SYNONYM emp FOR employees;
```

### Example 2: Replacing a Private Synonym

```
CREATE OR REPLACE SYNONYM emp FOR employees;
```

### Example 3: Creating a Public Synonym

```
CREATE PUBLIC SYNONYM dept FOR departments;
```

### Example 4: Replacing a Public Synonym

```
CREATE OR REPLACE PUBLIC SYNONYM dept FOR departments;
```

### Example 5: Querying After Creating a Synonym

```
-- Create a synonym
CREATE SYNONYM emp FOR employees;

-- Query using the synonym
SELECT * FROM emp WHERE department_id = 10;
```
