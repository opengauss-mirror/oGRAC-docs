# DROP USER

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:37:07.539Z pushedAt=2026-07-29T06:14:15.858Z -->

## Description

Drops a database user account.

## Precautions

-- Permission requirement: The operator who executes the user drop statement must have the `DROP USER` system privilege.

-- Existence check: If the user to be dropped does not exist and the `IF EXISTS` option is not specified in the statement, the system returns the error message "user name does not exist".

-- Operation restriction: This drop operation cannot be performed during database restart and rollback.

-- Prerequisite recommendation: The maximum number of objects associated with a single user is 50,000. To ensure smooth operation, it is advised to drop the user after dropping the table objects created by the user.

-- Risk warning: This operation must be performed with caution; even if the drop process is interrupted midway, the objects that have already been dropped cannot be recovered.

## Syntax

DROP USER [ IF EXISTS ] user_name [ CASCADE ]

## Parameter Description

- **user_name**: Name of the target user to be dropped.

- **IF EXISTS**: Used to verify the existence of the user to be dropped. If this option is specified, the statement returns success directly when the target user does not exist; if the user exists, the drop operation is performed.

- **CASCADE**: Option for handling associated objects when dropping a user:

  Without `CASCADE`: If the user to be dropped still owns any database objects, the system throws an error: `user objects is being used, can not drop`;
  With `CASCADE`: Drops a user while force-cascading the deletion of all database objects owned by the user, including: data tables, indexes, constraints (including foreign key constraints referencing the table), triggers,
             sequences, views (excluding objects owned by other users referenced in the views), functions/stored procedures, and user-owned tables in the recycle bin.

## Examples

Drop user `user001` and force-cascades the deletion of all database objects owned by this user.

-- Create user user001 and set the login password to "6fgfjFGgh".

CREATE USER user001 IDENTIFIED BY "6fgfjFGgh";

-- Drop the user user001 and forcibly remove all database objects owned by this user:

DROP USER user001 CASCADE;
