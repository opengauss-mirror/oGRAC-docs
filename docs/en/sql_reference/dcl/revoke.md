# REVOKE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:21.040Z pushedAt=2026-07-29T06:14:15.840Z -->

## Function Description

Revokes system permissions or roles from grantees.

## Precautions

-- Permission requirements for the executor of system permission revocation:
   When executing the REVOKE operation to revoke system permissions, the operator must meet one of the following conditions:
    The operator possesses the system permission to be revoked and has been granted the operation with the `WITH ADMIN OPTION` attribute;
    The operator possesses the `GRANT ANY PRIVILEGE` system permission.

-- Permission requirements for the executor of role revocation:
   When executing the REVOKE operation to revoke roles, the operator must meet one of the following conditions:
    The operator possesses the role and has been granted the operation with the `WITH ADMIN OPTION` attribute;
    The operator possesses the `GRANT ANY ROLE` system permission;
    The operator is the creator of the role to be revoked.

-- Permission existence check:
   If the target grantee does not possess the permission or role to be revoked, an error is reported when the revocation operation is executed.

-- DBA role permission restriction:
   Permissions of the DBA role cannot be revoked. The initial permissions of this role are determined during the database creation phase. Newly granted permissions can be added subsequently, but the already assigned permissions cannot be revoked.

## Syntax

-- Revoke system permissions:
   REVOKE { ALL [ PRIVILEGES ] | { system_privilege_name | role_name } [ , ... ] } FROM revokee;
   revokee clause: { user_name | role_name } [ , ... ]

-- Revoke object permissions:
   Revoke directory object permissions: REVOKE { READ } ON DIRECTORY dir_name FROM revokee;
     revokee clause: { user_name | role_name } [ , ... ]
   Revoke other object permissions: REVOKE { ALL [ PRIVILEGES ]|{ object_privilege_name [, ...] ON [object_type] [schema_name.]object_name } } FROM revokee;
     object_privilege_name clause: { SELECT | UPDATE | DELETE | INSERT | ALTER | INDEX | EXECUTE | READ | REFERENCES } [, ... ]
     object_type clause: [ TABLE | VIEW | SEQUENCE | PACKAGE | PROCEDURE | FUNCTION ]
     revokee clause: { user_name | role_name } [ , ... ]

-- Revoke the INHERIT PRIVILEGES permission of a user:
   REVOKE INHERIT PRIVILEGES ON USER user_name FROM revokee;
   revokee clause: { user_name } [ , ... ]

## Parameter Description

- `system_privilege_name`: name of a system permission. All system permissions currently supported by the system.

- `role_name`: name of a role. For details, see the description of the ROLE statement. When a role is revoked from a user or another role, all permissions under that role belonging to the revokee are removed.

- `ALL [ PRIVILEGES ]`: represents all system permissions. The `PRIVILEGES` keyword can be omitted.

- `object_privilege_name`: name of an object permission.

- `object_type`: Name of the object type. If this parameter is not specified, the system searches for objects in the following priority order: table, view, sequence, advanced package, stored procedure, function.

- `ALL [ PRIVILEGES ]`: Represents all object permissions. The `PRIVILEGES` keyword is optional.

- `[ schema_name. ]`: User name. If this parameter is not specified, the currently logged-in user is used by default.

- `revokee`: The grantee (the target from whom permissions are revoked), which can be a user or a role. Multiple revokees can be specified in a single operation.

- `user_name`: The name of the target user whose permissions are to be revoked.

## Examples

### Revoke System Permissions from User wangwu

--Delete user wangwu

DROP USER wangwu CASCADE;

--Create user wangwu

CREATE USER wangwu IDENTIFIED BY "8fgybjjFGgh";

--Grant the system permissions CREATE SESSION, CREATE TABLE, CREATE ANY INDEX, and CREATE USER to user wangwu.

GRANT CREATE SESSION, CREATE TABLE, CREATE ANY INDEX, CREATE USER TO wangwu;

--Revoke the system permission CREATE USER from the authorized user wangwu.

REVOKE CREATE USER FROM wangwu;

### Revoke a Role from User zhangfei

--Drop role role_test.

DROP ROLE role_test;

--Create role role_test.

CREATE ROLE role_test;

--Delete user zhangfei.

DROP USER zhangfei CASCADE;

--Create user zhangfei. The password is "2GHjkGHyv".

CREATE USER zhangfei IDENTIFIED BY "2GHjkGHyv";

--Delete user lvbu.

DROP USER lvbu CASCADE;

--Create user lvbu.

CREATE USER lvbu IDENTIFIED BY "fhghj78DC";

--Grant the system permissions CREATE SESSION, CREATE USER, CREATE ROLE, CREATE TABLE, CREATE ANY TABLE,
CREATE ANY INDEX, DROP USER, DROP ANY ROLE, DROP ANY TABLE, and DROP ANY INDEX to the role role_test.

GRANT CREATE SESSION, CREATE USER, CREATE ROLE, CREATE TABLE, CREATE ANY TABLE, CREATE
ANY INDEX, DROP USER, DROP ANY ROLE, DROP ANY TABLE, DROP ANY INDEX TO role_test;

-- Grant the role role_test to users zhangfei and lvbu.

GRANT role_test TO zhangfei, lvbu;

-- Revoke the role role_test from the authorized user zhangfei.

REVOKE role_test FROM zhangfei;
