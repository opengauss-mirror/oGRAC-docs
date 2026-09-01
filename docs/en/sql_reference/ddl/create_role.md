# CREATE ROLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:44.787Z pushedAt=2026-07-29T06:14:15.847Z -->

## Description

This statement is used to create a database role.

A role is a collection of permissions, including object privileges and system privileges. After database initialization is complete, the system automatically creates a set of predefined roles, which are essentially collections of permissions. The specific roles are as follows:

**DBA:**
This role cannot be deleted and has all system privileges.

**RESOURCE:**
This role has the privileges to create tables, sequences, stored procedures, functions, and triggers.

**CONNECT:**
This role has the privilege to connect to the database.

## Precautions

-- The relevant user must be granted the `CREATE ROLE` system privilege before executing this statement.
-- The role name must not duplicate any existing role name or user name in the database. Otherwise, an error will be reported.

## Syntax

CREATE ROLE role_name [ IDENTIFIED BY password [ ENCRYPTED ]]

## Parameter Description

- **role_name**: If the role name contains spaces or special characters other than #$_, the role name must be enclosed in backticks (``) or double quotation marks ("").

- **IDENTIFIED BY**: The created role uses a password. `IDENTIFIED BY` is followed by the specific password content.

- **password**: A reserved attribute that is not yet in use.

- **ENCRYPTED**: Indicates whether the specified password is encrypted. If it is encrypted, password specification verification is not required.

  Roles created using `ENCRYPTED` must log in with a plaintext password, so this method is not recommended for creating roles.

## Examples

-- Delete the role role_explorer.

DROP ROLE role_explorer;

-- Create the role role_explorer.

CREATE ROLE role_explorer;
