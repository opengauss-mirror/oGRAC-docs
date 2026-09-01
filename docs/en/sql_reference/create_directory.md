# CREATE DIRECTORY

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:14:32.182Z pushedAt=2026-08-17T01:16:41.782Z -->

## Function Description

Creates a directory object. A directory object points to a physical directory path in the OS, providing a secure way to manage interactions between the database and OS files. When creating a table, the `DIRECTORY` option can be specified to create an external table, allowing users to access external data using SQL without actually loading the data into the database.

## Precautions

- Creating a directory object requires the GRANT ANY DIRECTORY permission. Only the SYS user can own a directory object. Directory objects created by regular users belong to the SYS user.

- The path of a directory object is the path on the machine where the database program resides. Both relative and absolute paths are supported. The starting point of a relative path is the program's working directory.

- Before creating a directory object, you need to create the directory on the database node in advance. The owner and group of the directory object and its parent directory must be consistent with the database installation user, and the permissions must be 700.

- Since a directory object is valid only on the node where it is created, external tables cannot be accessed on other nodes.

## Syntax

**stmt:**

```sql
CREATE [OR REPLACE] DIRECTORY directory_name AS directory_path
```

## Parameter Description

- **OR REPLACE**: Replaces an existing directory object with the same name.

- **directory_name**: The name of the directory object, with a maximum length of 63 characters.

- **directory_path**: The path where the directory object resides. It must be enclosed in single quotation marks and has a maximum length of 187 characters.

## Examples

```
-- Create a directory object.
CREATE DIRECTORY data_dir_0 AS '/home/ogracdba/test';

-- Create or replace a directory object.
CREATE OR REPLACE DIRECTORY data_dir_0 AS '/home/ogracdba/test';

-- Create a directory object with a relative path.
CREATE DIRECTORY data_dir_1 AS './dir';

```
