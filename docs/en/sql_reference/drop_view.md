# DROP VIEW

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:22.663Z pushedAt=2026-08-17T09:30:53.734Z -->

## Function Description

Deletes one or more views from a database. When a view is deleted, its definition is removed from the data dictionary, but the data in the base tables is not affected.

## Precautions

When dropping a view that is depended on by other objects, it is recommended to use `CASCADE CONSTRAINTS` to handle dependent constraints, or use `IF EXISTS` to avoid errors when the view does not exist.

## Syntax

```sql
DROP VIEW [IF EXISTS] view_name [CASCADE CONSTRAINTS];
```

## Parameter Description

| Name | Description |
|------|------|
| IF EXISTS | Optional. If specified, the database does not raise an error but silently returns success when the view to be dropped does not exist. |
| view_name | Name of the view to be dropped. |
| CASCADE CONSTRAINTS | Optional. If specified, this parameter cascades the drop operation to all constraints that depend on this view. |

## Examples

### Basic Drop

```
-- Drop the view named employee_view
DROP VIEW employee_view;
```

### Using IF EXISTS

```
-- Drop the view if it exists; no error is reported if it does not exist.
DROP VIEW IF EXISTS employee_view;
```

### Cascading Drop of Constraints

```
-- Drop the view with cascade constraints that depend on it.
DROP VIEW employee_view CASCADE CONSTRAINTS;
```
