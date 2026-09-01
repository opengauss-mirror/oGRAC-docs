# GRANT

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:23.215Z pushedAt=2026-07-29T06:14:15.837Z -->

## Description

In database management, the `GRANT` command is used to assign system privileges or roles to users or other roles, enabling flexible privilege control.
If a user or role has not been granted privileges on `SYS` user objects, the user is not authorized to pass the privileges on `SYS` user objects to other users or roles.

## Precautions

(1) To grant a system privilege, the user must meet one of the following conditions before executing the relevant statement:
-- The user has been granted the relevant system privilege and has the `WITH ADMIN OPTION` attribute.
-- The user has been granted the system privilege GRANT ANY PRIVILEGE.

(2) To grant a role, the user must meet one of the following conditions before executing the relevant statement:
-- The user has been granted the relevant role and has the `WITH ADMIN OPTION` attribute.
-- The user has been granted the system privilege GRANT ANY ROLE.
-- The user is the creator of the relevant role.

(3) When a user attempts to access an object owned by another user, the database first checks whether the object exists under the accessed user:
-- If the object does not exist, an error indicating that the object does not exist is returned.

(4) An object owner has full control over the objects under their ownership by default, including the ability to grant privileges on these objects to other users.

(5) The maximum number of object privileges that can be granted to a single user is 2048 * 2048. When a user receives the full set of object privileges from another user, privilege loading is required during database startup, which will prolong the startup time.

> **NOTE:** The following privilege grant operations are high-risk and must be performed with caution:
>
> - CREATE ANY TABLE
> - CREATE ANY INDEX
> - CREATE ANY SEQUENCE
> - CREATE ANY VIEW
> - CREATE ANY SYNONYM
> - CREATE ANY PROCEDURE
> - DROP ANY TABLE
> - DROP ANY INDEX
> - DROP ANY SEQUENCE
> - DROP ANY VIEW
> - DROP ANY SYNONYM
> - LOCK ANY TABLE
> - DROP ANY PROCEDURE
> - ALTER ANY TABLE
> - ALTER ANY INDEX
> - ALTER ANY SEQUENCE
> - UPDATE ANY TABLE
> - INSERT ANY TABLE
> - DELETE ANY TABLE
> - ANALYZE ANY
> - ALTER ANY TRIGGER
> - CREATE ANY TRIGGER
> - DROP ANY TRIGGER
> - EXECUTE ANY PROCEDURE
> - CREATE ANY TYPE
> - EXECUTE ANY TYPE
> - DROP ANY TYPE
> - SYSDBA
> - GRANT ANY PRIVILEGE
> - GRANT ANY ROLE
> - EXEMPT REDACTION POLICY
> - EXEMPT ACCESS POLICY
> - SELECT ANY DICTIONARY
> - ALTER USER
> - DROP USER

## Syntax

GRANT { ALL [ PRIVILEGES ] |{ system_privilege_name | role_name } [ , ... ] } TO grantee [ WITH ADMIN OPTION ]

-- grantee clause:
{ user_name | role_name } [ , ... ]

GRANT { object_privilege_name | ALL [PRIVILEGES] } [, ...] ON [object_type] [schema_name.]object_name
TO grantee [ WITH GRANT OPTION ]

-- object_privilege_name clause:
{ SELECT | UPDATE | DELETE | INSERT | ALTER | INDEX | EXECUTE | READ | REFERENCES } [, ... ]

-- object_type clause
[ TABLE | VIEW | SEQUENCE | PACKAGE | PROCEDURE | FUNCTION ]

-- grantee clause:
{ user_name | role_name } [ , ... ]
GRANT { READ } ON DIRECTORY dir_name TO grantee [ WITH GRANT OPTION ]

-- grantee clause
{ user_name | role_name } [ , ... ]

## Parameter Description

- **system_privilege_name**: System privilege name. The following table shows the system privileges currently supported by the system and the privilege ownership of roles or users, where "T" indicates that the privilege is owned and "F" indicates that it is not.

  **System Privilege Table**

  |System Privilege|||Granted to Role|||Granted to User||
  |---|---|---|---|---|---|---|---|
  |Operation|Privilege|Description|DBA|RESOURCE|CONNECT|SYS|PUBLIC|
  |Create session|CREATE SESSION|New users need to be granted this privilege to connect to the database.|T|F|T|T|F|
  |Create user|CREATE USER|This privilege is required to create users.|T|F|F|T|F|
  |Create role|CREATE ROLE|This privilege is required to create roles.|T|F|F|T|F|
  |Create tablespace|CREATE TABLESPACE|This privilege is required to create tablespaces.|T|F|F|T|F|
  |Create table|CREATE TABLE|This privilege is required to create tables in the current schema.|T|T|F|T|F|
  |Create any table|CREATE ANY TABLE|This privilege is required to create tables in other schemas.|T|F|F|T|F|
  |Create index|CREATE ANY INDEX|Create an index in the current schema if one of the following conditions is met: the indexed table is in the current schema. The CREATE ANY INDEX privilege is required if creating an index in another schema.|T|F|F|T|F|
  |Create sequence|CREATE SEQUENCE|This privilege is required to create sequences in the current schema.|T|T|F|T|F|
  |Create any sequence|CREATE ANY SEQUENCE|This privilege is required to create sequences in other schemas.|T|F|F|T|F|
  |Create view|CREATE VIEW|This privilege is required to create views in the current schema.|T|F|F|T|F|
  |Create any view|CREATE ANY VIEW|This privilege is required to create views in other schemas. For ordinary users creating views based on SYS tables, additional privileges are required: if ENABLE_ACCESS_DC=FALSE, the SELECT object privilege on SYS tables is required; otherwise, the SELECT privilege (object privilege or system privilege) on SYS tables is required.|T|F|F|T|F|
  |Create control file|CREATE CTRLFILE|In the nomount state, the `SYS` user can create a control file when the file is missing. Duplicate creation is not allowed.|T|F|F|T|F|
  |Create database instance|CREATE DATABASE|In the nomount state, the `SYS` user can create a database instance. Duplicate creation is not allowed.|T|F|F|T|F|
  |Create database link|CREATE DATABASE LINK|This privilege is required to create database links.|T|F|F|T|F|
  |Create synonym|CREATE SYNONYM, CREATE ANY SYNONYM, CREATE PUBLIC SYNONYM|To create a synonym in the current schema, any one of the three privileges is sufficient. To create a synonym in another schema, the CREATE ANY SYNONYM privilege is required. To create a public synonym, the CREATE PUBLIC SYNONYM privilege is required.|T|F|F|T|F|
  |Create function/stored procedure/user-defined advanced package|CREATE PROCEDURE|This privilege is required to create functions/stored procedures/user-defined advanced packages in the current schema.|T|T|F|T|F|
  |Create any function/stored procedure/user-defined advanced package|CREATE ANY PROCEDURE|This privilege is required to create functions/stored procedures/user-defined advanced packages in other schemas.|T|F|F|T|F|
  |Drop user|DROP USER|This privilege is required to drop users.|T|F|F|T|F|
  |Drop tablespace|DROP TABLESPACE|This privilege is required to drop tablespaces.|T|F|F|T|F|
  |Drop table|DROP ANY TABLE|By default, users can drop tables in their own schema. This privilege is required to drop tables owned by other users.|T|F|F|T|F|
  |Drop index|DROP ANY INDEX|By default, users can drop indexes in their own schema. This privilege is required to drop indexes owned by other users.|T|F|F|T|F|
  |Drop sequence|DROP ANY SEQUENCE|By default, users can drop sequences in their own schema. This privilege is required to drop sequences owned by other users.|T|F|F|T|F|
  |Drop view|DROP ANY VIEW|By default, users can drop views in their own schema. This privilege is required to drop views owned by other users.|T|F|F|T|F|
  |Drop synonym|DROP ANY SYNONYM, DROP PUBLIC SYNONYM|By default, users can drop synonyms in their own schema. The DROP ANY SYNONYM privilege is required to drop synonyms owned by other users, and the DROP PUBLIC SYNONYM privilege is required to drop public synonyms. The system only supports dropping public synonyms.|T|F|F|T|F|
  |Drop role|DROP ANY ROLE|By default, users can drop roles in their own schema. This privilege is required to drop roles owned by other users, or the user must have been granted the role with WITH GRANT OPTION.|T|F|F|T|F|
  |Drop database link|DROP DATABASE LINK|This privilege is required to drop database links.|T|F|F|T|F|
  |Lock table|LOCK ANY TABLE|By default, users can lock tables in their own schema. This privilege is required to lock tables owned by other users.|T|F|F|T|F|
  |Truncate table|DROP ANY TABLE|By default, users can delete all rows from tables in their own schema using TRUNCATE TABLE. This privilege is required to delete all rows from tables owned by other users.|T|F|F|T|F|
  |Flashback table|FLASHBACK ANY TABLE|To flash back a table in the current schema, the required privilege is the same as DROP TABLE. This privilege is required to flash back tables owned by other users. Note: Different privileges will be required based on the synchronization type (SCN/TIMESTAMP/BEFORE) of the FLASHBACK TABLE operation in the future.|T|F|F|T|F|
  |Purge operation|DROP ANY TABLE, DROP ANY INDEX, DROP TABLESPACE, PURGE DBA_RECYCLEBIN|PURGE supports: tables, indexes, tablespaces, and the recycle bin. By default, users can purge objects in their own schema. The corresponding privilege is required to purge objects owned by other users.|T|F|F|T|F|
  |Drop function/stored procedure/user-defined advanced package|DROP ANY PROCEDURE|By default, users can drop functions/stored procedures/user-defined advanced packages in their own schema. This privilege is required to drop those owned by other users.|T|F|F|T|F|
  |Modify session parameters|ALTER SESSION|The current session has the privilege to modify all parameters by default. This privilege is reserved.|T|F|F|T|F|
  |Modify tablespace|ALTER TABLESPACE|This privilege is required to modify tablespaces.|T|F|F|T|F|
  |Modify system attributes|ALTER SYSTEM|This privilege is required to modify system attributes.|T|F|F|T|F|
  |Modify database|ALTER DATABASE|This privilege is required to modify database attributes.|T|F|F|T|F|
  |Modify database link|ALTER DATABASE LINK|This privilege is required to modify database links.|T|F|F|T|F|
  |Modify user information|ALTER USER|By default, users can modify their own passwords. This privilege is required to modify passwords of other users.|T|F|F|T|F|
  |Modify table|ALTER ANY TABLE|By default, users can modify tables in their own schema. This privilege or the ALTER object privilege on the table is required to modify tables owned by other users.|T|F|F|T|F|
  |Modify index|ALTER ANY INDEX|By default, users can modify the attributes of indexes in their own schema. This privilege is required to modify index attributes owned by other users.|T|F|F|T|F|
  |Modify sequence|ALTER ANY SEQUENCE|By default, users can modify sequences in their own schema. This privilege or the ALTER object privilege on the sequence is required to modify sequences owned by other users.|T|F|F|T|F|
  |Access sequence|SELECT ANY SEQUENCE|By default, users can access sequences in their own schema. This privilege is required to access sequences owned by other users.|T|F|F|T|F|
  |Grant operation|GRANT ANY PRIVILEGE, GRANT ANY ROLE, GRANT ANY OBJECT PRIVILEGE|When granting a system privilege, the user executing the grant statement must meet one of the following conditions: has been granted the system privilege with the `WITH ADMIN OPTION` attribute; or has been granted the GRANT ANY PRIVILEGE system privilege. When granting a role, the user executing the grant statement must meet one of the following conditions: has been granted the role with the `WITH ADMIN OPTION` attribute; or has been granted the GRANT ANY ROLE system privilege; or is the creator of the role. Users with the GRANT ANY OBJECT PRIVILEGE system privilege can grant object privileges on objects under any user name to other users.|T|F|F|T|F|
  |Revoke operation|GRANT ANY PRIVILEGE, GRANT ANY ROLE|The user executing a revoke system privilege statement must meet one of the following conditions: has been granted the system privilege with the `WITH ADMIN OPTION` attribute; or has been granted the GRANT ANY PRIVILEGE system privilege. The user executing a revoke role statement must meet one of the following conditions: has been granted the role with the `WITH ADMIN OPTION` attribute; or has been granted the GRANT ANY ROLE system privilege; or is the creator of the role.|T|F|F|T|F|
  |Modify PROFILE attributes|ALTER PROFILE|This privilege is required to modify profile attributes.|T|F|F|T|F|
  |Create PROFILE|CREATE PROFILE|This privilege is required to create profiles.|T|F|F|T|F|
  |Drop PROFILE|DROP PROFILE|This privilege is required to drop profiles.|T|F|F|T|F|
  |Read table or view data of `SYS` user|SELECT ANY DICTIONARY|This privilege is required to query tables or views of the `SYS` user.|T|F|F|T|F|
  |Read any table or view data|READ ANY TABLE, SELECT ANY TABLE|Users with this privilege can query data from any table or view. Access to SYS tables is controlled by the parameter ENABLE_ACCESS_DC (default TRUE; modifiable only by the `SYS` user; the DBA role is not restricted by ENABLE_ACCESS_DC). ENABLE_ACCESS_DC=TRUE → SELECT ANY TABLE can access `SYS` objects. ENABLE_ACCESS_DC=FALSE → SELECT ANY TABLE cannot access `SYS` objects. Queries containing the FOR UPDATE clause are limited to users with the SELECT ANY TABLE privilege.|T|F|F|T|F|
  |Comment privilege|COMMENT ANY TABLE|By default, users can comment on tables in their own schema. This privilege is required to comment on tables owned by other users.|T|F|F|T|F|
  |Update table data|UPDATE ANY TABLE|This privilege is required to update data in any table.|T|F|F|T|F|
  |Insert table data|INSERT ANY TABLE|This privilege is required to insert data into any table.|T|F|F|T|F|
  |Delete table data|DELETE ANY TABLE|This privilege is required to delete data from any table.|T|F|F|T|F|
  |Backup operation|SYSBACKUP, SYSDBA|This privilege is required to perform backup operations. When ENABLE_SYSDBA_REMOTE_LOGIN is enabled, users with the SYSDBA privilege can log in using AS SYSDBA. After login, the user is elevated to the `SYS` user and obtains all privileges of the `SYS` user.|T|F|F|T|F|
  |Restore operation|SYSDBA|This privilege is required to perform restore operations.|T|F|F|T|F|
  |Stop database|SYSDBA|This privilege is required to stop the database.|T|F|F|T|F|
  |Rebuild standby|SYSDBA|This privilege is required to rebuild the standby server.|T|F|F|T|F|
  |Validate physical backup set|SYSDBA|This privilege is required to validate the integrity of physical backup sets.|T|F|F|T|F|
  |Analyze table operation|ANALYZE ANY|By default, users can analyze tables in their own schema. This privilege is required to analyze tables owned by other users (except the `SYS` user). The DBA privilege is required to analyze `SYS` user tables.|T|F|F|T|F|
  |Modify trigger|ALTER ANY TRIGGER|By default, users can modify triggers in their own schema. This privilege is required to modify triggers owned by other users.|T|F|F|T|F|
  |Create trigger|CREATE TRIGGER|This privilege is required to create triggers belonging to the user.|T|T|F|T|F|
  |Create trigger|CREATE ANY TRIGGER|This privilege is required to create triggers belonging to other users.|T|F|F|T|F|
  |Drop trigger|DROP ANY TRIGGER|By default, users can drop triggers in their own schema. This privilege is required to drop triggers owned by other users.|T|F|F|T|F|
  |Execute function/stored procedure/user-defined advanced package|EXECUTE ANY PROCEDURE|By default, users can execute functions/stored procedures/user-defined advanced packages in their own schema. This privilege or the EXECUTE object privilege on the specified function/stored procedure/user-defined advanced package is required to execute those owned by other users.|T|F|F|T|F|
  |Modify stored procedure|ALTER ANY PROCEDURE|This privilege is reserved.|T|F|F|T|F|
  |Modify materialized view|ALTER ANY MATERIALIZED VIEW|This privilege is reserved.|T|F|F|T|F|
  |Create materialized view|CREATE ANY MATERIALIZED VIEW, CREATE MATERIALIZED VIEW|This privilege is reserved.|T|F|F|T|F|
  |Drop materialized view|DROP ANY MATERIALIZED VIEW|This privilege is reserved.|T|F|F|T|F|
  |Modify role attributes|ALTER ANY ROLE|This privilege is reserved.|T|F|F|T|F|
  |Flashback archive|FLASHBACK ARCHIVE ADMINISTER|This privilege is reserved.|T|F|F|T|F|
  |Global query rewrite|GLOBAL QUERY REWRITE|This privilege is reserved.|T|F|F|T|F|
  |Manage tablespace|MANAGE TABLESPACE|This privilege is reserved.|T|F|F|T|F|
  |Materialized view refresh on commit|ON COMMIT REFRESH|This privilege is reserved.|T|F|F|T|F|
  |Unlimited tablespace usage|UNLIMITED TABLESPACE|This privilege is reserved.|T|F|F|T|F|
  |Create subview|UNDER ANY VIEW|This privilege is reserved.|T|F|F|T|F|
  |System operator privilege|SYSOPER|This privilege is reserved.|T|F|F|T|F|
  |Create user-defined type|CREATE TYPE, CREATE ANY TYPE|The CREATE TYPE privilege is required to create user-defined types in the current schema. The CREATE ANY TYPE privilege is required to create user-defined types in other schemas.|T|F|F|T|F|
  |Use user-defined type|EXECUTE ANY TYPE|By default, users can use user-defined types in their own schema. This privilege or the EXECUTE object privilege on the specified type is required to use user-defined types owned by other users.|T|F|F|T|F|
  |Drop user-defined type|DROP ANY TYPE|By default, users can drop user-defined types in their own schema. This privilege is required to drop user-defined types owned by other users.|T|F|F|T|F|
  |Redaction policy exemption|EXEMPT REDACTION POLICY|Users with this privilege are exempt from redaction policies and can view the original data.|T|F|F|T|F|
  |Invoker privilege|INHERIT ANY PRIVILEGES|For users with the INHERIT ANY PRIVILEGES privilege, all objects under their name (including triggers, stored procedures, user-defined functions, and user-defined advanced packages) are considered by all other users as not containing risky operations.|F|F|F|T|F|
  |Security policy exemption|EXEMPT ACCESS POLICY|Users with the EXEMPT ACCESS POLICY privilege are exempt from security policy restrictions.|T|F|F|T|F|
  |Force transaction|FORCE ANY TRANSACTION|Users with the FORCE ANY TRANSACTION privilege can perform forced commit or forced rollback operations on uncompleted transactions initiated by other users.|T|F|F|T|F|
  |Use tablespace|USE ANY TABLESPACE|Users with the USE ANY TABLESPACE privilege can specify the SYSTEM or SYSAUX tablespace when creating database objects such as tables and indexes.|T|F|F|T|F|

  **Explanation**

  -- The `SYS` user has all the privileges described above by default.

  1. Synonym privilege rules

  The privileges of a synonym are exactly the same as those of the object it points to:
  (1) Granting a synonym privilege to a user is equivalent to granting the privilege of the object corresponding to the synonym to that user.
  (2) Conversely, when granting an object privilege to a user, if the object has a synonym, the privilege of the synonym is also granted to that user simultaneously.
  (3) When accessing a synonym, the system checks whether the current operating user has the privilege of the object corresponding to the synonym.

  2. Role privilege inheritance rules

  (1) Role granting: After a role is granted to a user or another role, the user or role inherits all privileges of that role (including system privileges and all granted object privileges).
  (2) Role revocation: When a role is revoked from a user or role, the privileges inherited through that role are revoked. If the same privilege can be inherited from another role, the user or role retains that privilege after the role is revoked.

  3. Privilege linkage rules for object/user/role deletion

  (1) Object deletion: When a table, view, sequence, or stored procedure is deleted, the privileges on that object held by users or roles are also deleted synchronously.
  (2) User deletion: When a user is deleted, all objects owned by that user are deleted together, and the privileges on these objects granted to other users are also revoked synchronously.
  (3) Role deletion: When a role is deleted, all privileges granted to that role are cleared, and the corresponding privileges of users or roles to which the role was granted are also revoked synchronously.

  4. Privilege requirements for view creation and access

  (1) View creation:
  Basic privilege: The current user must have the CREATE VIEW (create own view) or CREATE ANY VIEW (create any user's view) system privilege.
  Dependency privilege: The view owner (which may be the current user or another user) must have the SELECT privilege on the objects referenced in the view.
  (2) View access:
  Basic privilege: The user must have the SELECT or SELECT ANY TABLE privilege, or the READ or READ ANY TABLE privilege on the view.
  Dependency privilege: The view owner must have the SELECT privilege on the objects referenced in the view.

  5. Privilege rules for stored procedures, user-defined functions, and triggers

  (1) Creation phase: When creating a stored procedure, user-defined function, or trigger, access privileges on dependent objects are not checked (a warning is issued if privileges are insufficient).
  (2) Execution phase: When executing a stored procedure, calling a user-defined function, or when a trigger fires, the current user's access privileges on dependent objects are checked.
  (3) Cross-user trigger creation: If a trigger is created on another user's table, the creator must be granted the CREATE ANY TRIGGER privilege.

  6. General check rules for object privileges

  The object owner has all privileges on the object by default. The privilege requirements for different operations are as follows:
  (1) SELECT: Accessing one's own objects requires no authorization; accessing another user's objects requires the SELECT privilege on the object or the SELECT ANY TABLE system privilege.
  (2) UPDATE: Updating one's own objects requires no authorization; updating another user's objects requires the UPDATE privilege on the object or the UPDATE ANY TABLE system privilege.
  (3) DELETE: Deleting data from one's own objects requires no authorization; deleting data from another user's objects requires the DELETE privilege on the object or the DELETE ANY TABLE system privilege.
  (4) INSERT: Inserting data into one's own objects requires no authorization; inserting data into another user's objects requires the INSERT privilege on the object or the INSERT ANY TABLE system privilege.
  (5) ALTER: Modifying the definition of one's own objects requires no authorization; modifying another user's objects requires the ALTER privilege on the object or the ALTER ANY TABLE system privilege.
  (6) INDEX: Creating an index requires the INDEX privilege on the object.
  (7) EXECUTE:
  Executing a stored procedure or calling a user-defined function/system package function requires the EXECUTE privilege on the object.
  (8) READ: The privilege to access an object, functionally similar to the SELECT privilege. The difference is that the SELECT privilege is required when a statement contains a FOR UPDATE clause, while SELECT and READ are equivalent when no such clause is present.
  (9) REFERENCES: Creating a foreign key constraint requires the REFERENCES privilege on the referenced object.

-- Even if a non-SYS user holds the relevant privileges, they cannot create objects in the `SYS` user's schema, nor can they perform operations on the `SYS` user's objects. Only when the `SYS` user separately grants the corresponding object privileges to a non-SYS user can the latter operate on `SYS` objects.

For details about the list of high-risk privileges, see "Precautions" above.

- **role_name**: -- Description of role authorization rules

  (1) Role definition and authorization effect: For descriptions related to role_name, see the ROLE statement documentation. When a role is granted to a user or another role, the grantee inherits all system privileges contained in that role.
  (2) Authorization restriction: Circular role granting is prohibited. For example, the following scenario is forbidden: granting roleA to roleB, roleB to roleC, and roleC to roleA.
  (3) System predefined roles: The system includes the following roles by default: dba, resource, and connect. Among them, the DBA role possesses all system privileges. Exercise caution when granting this role to ordinary users.

- **ALL [ PRIVILEGES ]**: This keyword represents all system privileges and object privileges. The PRIVILEGES keyword is optional.

- **[schema_name.]**: This parameter represents the target username. If not explicitly specified, it defaults to the currently logged-in user.

- **[WITH ADMIN OPTION]**: -- The specific meanings of this option are as follows

  (1) Privilege delegation: The grantee obtains the permission to further grant the system privilege or role to other users or roles (i.e., enabling the transfer of grant privileges). When the privilege is revoked, the privileges previously granted by the grantee to other users are not cascadingly revoked.
  (2) Self-revocation: The grantee may voluntarily revoke the system privilege or role they hold.
  (3) Operation restriction: Once this option is set, only the entire privilege can be revoked via the REVOKE statement. It is not possible to revoke only this option while retaining the basic privilege.

- **object_privilege_name**: Name of an object privilege. The set of privileges varies depending on the type of database object. Currently supported object types include: tables, views, sequences, stored procedures, functions, triggers, and user-defined types.

  For details about the operation privileges supported by each object type, see the following table. The identifiers in the table are described as follows: T: The object supports this privilege. F: The object does not support this privilege. This privilege is reserved: This privilege is reserved for the object.

  > **NOTE:** Object owners and system administrators (`SYS` users and the DBA role) have all privileges on all objects in the following table by default.

  **Object Privilege Mapping Table**

  |Object Type|SELECT|UPDATE|INSERT|DELETE|INDEX|READ|REFERENCES|ALTER|EXECUTE|WRITE|
  |---|---|---|---|---|---|---|---|---|---|---|
  |Table|T|T|T|T|T|T|T|T|F|F|
  |View|T|This privilege is reserved|This privilege is reserved|This privilege is reserved|F|T|This privilege is reserved|F|F|F|
  |Sequence|T|F|F|F|F|F|F|T|F|F|
  |Stored procedure|F|F|F|F|F|F|F|F|T|F|
  |Trigger|F|F|F|F|F|F|F|F|T|F|
  |Function|F|F|F|F|F|F|F|F|T|F|
  |User-defined type|F|F|F|F|F|F|F|F|T|F|

  -- When GRANT ALL PRIVILEGES is used to grant object privileges to a user, the system determines the set of privileges to be granted or revoked based on the object type.

  -- If an object privilege is granted to PUBLIC, all users in the system (including users added after the authorization operation) will obtain the corresponding privilege on that object. This operation must be performed with caution. You can query the default privileges of the PUBLIC user after database installation using the following SQL statement:
  select * from ADM_TAB_PRIVS where GRANTEE='PUBLIC';

- **object_type**: The object type name. If this parameter is not explicitly specified, the system searches for the corresponding object in the priority order of table → view → sequence → stored procedure → function.

- **grantee**: -- Basic rules

  grantee refers to the grantee of the privilege, which can be specified as a user or role. If grantee is specified as PUBLIC, it means granting the privilege to all users in the system (including users added after the authorization operation). This operation must be performed with caution.

- **WITH GRANT OPTION**: -- The grantee obtains the privilege delegation capability, allowing them to grant this privilege to other users or roles.

  -- Revoking the privilege triggers a cascade effect, meaning that when the privilege is revoked from this user, any privileges they granted to other principals through this option are also revoked simultaneously.

  -- Once configured, this option can only be removed by revoking the corresponding privilege entirely through the REVOKE statement. It cannot be canceled independently while retaining the basic privilege.

## Examples

-- Create user songjiang and grant the CREATE SESSION privilege to user songjiang.

-- Delete user songjiang.

DROP USER songjiang CASCADE;

-- Create user songjiang with the password "ghQMO789".

CREATE USER songjiang IDENTIFIED BY "ghQMO789";

-- Grant the CREATE SESSION privilege to user songjiang.

GRANT CREATE SESSION TO songjiang;

-- Create user wusong, user wukong, and role bajie, and grant role bajie to user wusong and user wukong.

-- Drop role bajie.

DROP ROLE bajie;

-- Create role bajie.

CREATE ROLE bajie;

-- Grant the CREATE SESSION, CREATE USER, CREATE ROLE, and CREATE TABLE privileges to role bajie.

GRANT CREATE SESSION, CREATE USER, CREATE ROLE, CREATE TABLE TO bajie;

-- Drop user wusong.

DROP USER wusong CASCADE;

-- Create user wusong, with the password "GHPihb570".

CREATE USER wusong IDENTIFIED BY "GHPihb570";

-- Drop user wukong.

DROP USER wukong CASCADE;

-- Create user wukong with the password "XBYgh450".

CREATE USER wukong IDENTIFIED BY "XBYgh450";

-- Grant the role bajie to users wusong and wukong.

GRANT bajie TO wusong, wukong;

-- Create the table officer, create the user tangseng, and grant the user tangseng the INSERT privilege on this table.

-- Drop the table officer.

DROP TABLE IF EXISTS officer;

-- Create the table officer.

CREATE TABLE officer
(office_id NUMBER(8),
first_name VARCHAR2(20),
last_name VARCHAR2(25) CONSTRAINT off_last_name_nn NOT NULL,
email VARCHAR2(25) CONSTRAINT off_email_nn NOT NULL,
monile_no VARCHAR2(20),
hire_date DATE DEFAULT SYSDATE CONSTRAINT off_hire_date_nn NOT NULL,
job_id VARCHAR2(10) CONSTRAINT off_job_nn NOT NULL,
salary NUMBER(10,2) CONSTRAINT off_salary_nn NOT NULL,
pct NUMBER(4,2),
manager_id NUMBER(8),
dept_id NUMBER(6),
dn VARCHAR2(300),
CONSTRAINT off_email_uk UNIQUE (email)
) ;

-- Delete the user tangseng.

DROP USER tangseng CASCADE;

-- Create the user tangseng with the password "minBY78fg".

CREATE USER tangseng IDENTIFIED BY "minBY78fg";

-- Grant the user tangseng the INSERT privilege on the table officer.

-- Grant user tangseng the INSERT privilege on table officer.
GRANT INSERT ON officer TO tangseng;
