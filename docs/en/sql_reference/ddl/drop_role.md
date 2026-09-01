# DROP ROLE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:37:00.792Z pushedAt=2026-07-29T06:14:15.850Z -->

## Description

When a database role is dropped, all privileges owned by the role are revoked synchronously after the drop operation, and these privileges are also revoked from the users or other roles that have been granted the role. If the role does not exist, the system throws an error.

## Precautions

-- The user executing this statement must meet one of the following privilege conditions: having the `DROP ANY ROLE` system privilege; being the creator of the role; or having been granted the role with the `WITH ADMIN OPTION` attribute.
-- For roles created by system users, only system users have the privilege to drop such roles.
-- The name of the role to be dropped must exist in the database; otherwise, the operation triggers an error.
-- This operation is not supported during database restart and rollback.

## Syntax

DROP ROLE role_name

## Parameter Description

- **role_name**: Role name.

## Examples

-- Create the role role_explorer.

CREATE ROLE role_explorer;

-- Drop the role role_explorer.

DROP ROLE role_explorer;
