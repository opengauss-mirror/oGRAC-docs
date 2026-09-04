# COMMENT ON

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:58.845Z pushedAt=2026-08-25T01:50:15.402Z -->

## Function Description

Adds a comment to a table, view, or column. The added comments can be viewed through the following system views:

- **Table/View comments**: `MY_TAB_COMMENTS` and `ADM_TAB_COMMENTS`

- **Column comments**: `MY_COL_COMMENTS` and `ADM_COL_COMMENTS`

## Precautions

- **Permission requirements**:

  - No additional permissions are required to add comments to your own tables.

  - The `COMMENT ANY TABLE` permission is required to add comments to other users' tables.

- **Supported scenarios**:

  - Adding column comments during table creation is supported.

- **Restrictions**:

  - This operation is not supported during database restart or rollback.

## Syntax

```sql
COMMENT ON 
{
    TABLE [ schema_name. ] { table_name | view_name }
  | COLUMN [ schema_name. ] { table_name. | view_name. } column_name
} 
IS 'string';
```

## Parameter Description

- `[ schema_name. ]`: Username (schema name), which defaults to the currently logged-in user.

- `{ table_name | view_name }`: Name of the table or view to which the comment is added.

- `[ schema_name. ] { table_name. | view_name. } column_name`: Name of the column to which the comment is added.

- `IS`: Keyword that specifies the comment content.

- `string`: Comment text, with a maximum length of 4,000 bytes.

## Examples

```
-- Delete the user table user_info (if exists).
DROP TABLE IF EXISTS user_info;

-- Create a user table user_info.
CREATE TABLE user_info (user_id INT PRIMARY KEY,username VARCHAR(50) NOT NULL, email VARCHAR(100), create_time DATETIME DEFAULT CURRENT_TIMESTAMP, status CHAR(1) DEFAULT 'A');

-- Add a comment for the table user_info.
COMMENT ON TABLE user_info IS 'table of user info';

-- Add a comment for the column user_id.
COMMENT ON COLUMN user_info.user_id IS 'id of users';
```