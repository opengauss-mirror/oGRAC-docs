# CREATE USER

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:59.937Z pushedAt=2026-07-30T01:12:08.711Z -->

## Description

Creates a database user account.

## Notes

-- Permission requirements: The operator executing the user creation statement must have the `CREATE USER` system permission.

-- Quantity limit: The database supports a maximum of 15,000 users. Exceeding this limit triggers an error message.

-- Permission capacity: A single user can hold object permissions for up to 2048 × 2048 objects.

-- Name uniqueness: The username must not duplicate any existing username or role name in the database. Otherwise, the system returns the error message: "user *name* already exists".

-- When creating a user, a username and an initial password must be specified. The user is required to change the password upon first login.

-- When creating a new user through the ogsql client, to enhance security, password expiration can be manually set during the creation process, or configured later using the `ALTER USER` statement. The user must then log in through the ogsql client and change the password interactively, thereby controlling the scope of password disclosure.

-- Restrictions on system-preset users: `SYS` and `PUBLIC` are built-in system users and must not be created manually:
   (1) `SYS` user: Used for the initial creation process of a database instance.
   (2) `PUBLIC` user: A system-preset public user that represents the set of all database users. If a permission is granted to the `PUBLIC` user, that permission applies to all database users. Examples are as follows:
        ▪ Granting the `SELECT` permission on the user.t table to the `PUBLIC` user allows all users to access the table: `GRANT SELECT ON user.t TO PUBLIC`;
        ▪ Granting the `DBA` permission to the `PUBLIC` user grants the `DBA` permission to all users: `GRANT DBA TO PUBLIC`;

-- Permission inheritance control: When the `AUTO_INHERIT_USER` parameter is set to `ON`, the `INHERIT PRIVILEGES` permission of a newly created user is granted to the public user by default. To ensure database security, granting this permission to the public user is prohibited.

-- Audit log rule: The password specified in the `CREATE USER` statement is replaced with "*" in the audit log for recording purposes.

## Syntax

CREATE USER user_name
  IDENTIFIED BY password
  [ ENCRYPTED

    | DEFAULT TABLESPACE tablespace_name
    | TEMPORARY TABLESPACE tablespace_name
    | PASSWORD EXPIRE
    | ACCOUNT { LOCK | UNLOCK }
    | PROFILE profile_name
    | PERMANENT

  ] [ ... ]

## Parameter Description

- **user_name (Username)**:

  - Length limit: The username length must satisfy 1 ≤ character count ≤ 64.

  - First character rule: A digit is not allowed as the first character of a username.

  - Special character prohibition: The username must not contain the following characters (even when enclosed in double quotation marks or backticks): semicolon (;), vertical bar (|), backtick (`), dollar sign ($), ampersand (&), greater-than sign (>), less-than sign (<), double quotation mark ("), single quotation mark ('), exclamation mark (!), space, copyright symbol (©).

  - Special character compliance: If the username contains special characters other than the prohibited ones listed above, it must be enclosed in double quotation marks ("") or backticks (``).

  - Format conversion: Usernames are automatically converted to uppercase by default. Leading Tab characters within double quotation marks are automatically stripped, and the final stored username contains no spaces.

  - Keyword avoidance: `SYSDBA` and `CLSMGR` are database keywords. Creating a user with such names will result in login failure, so it is advised to avoid using them.

- **IDENTIFIED BY**: Password specification clause, used to set an initial password for the user upon creation.

- **password**: User password.

  The password must comply with the following rules:

  - **Length**: The length of a regular password must be no less than the minimum value set by the `PASSWORD_MIN_LEN` parameter in the `PROFILE`, and no more than 64 characters.

  - **First Character**: If the password is not enclosed in quotation marks, the first character must be a letter, a `#` sign, or an underscore.

  - **Uniqueness**: The password must not be identical to the username (or the reversed username). The check is case-insensitive.

  - **Complexity**: The password may contain only the following four categories of characters: digits, lowercase letters, uppercase letters, and spaces or special characters. It must include at least three of these four categories.

  - **Quoting**: If the password contains special characters other than _#$ or spaces, it must be enclosed in quotation marks.

  - **Login Adaptation**: If the password contains the special character `$`, when connecting via `ogsql`, if the password is enclosed in single quotation marks, no `\` is required; if it is not enclosed in single quotation marks, `\` is required (otherwise, the login will fail).

  - **Password Containing Spaces**: Only interactive login is supported.

  - **Login via conn Command**: All characters in the password segment are treated as the password itself.

- **DEFAULT TABLESPACE tablespace_name**: Defines the user's private default tablespace. The specified tablespace must already exist.

- **TEMPORARY TABLESPACE tablespace_name**: Defines the default temporary tablespace for the user. The specified tablespace must already exist. Currently, only the temp tablespace is supported as the temporary tablespace.

- **PASSWORD EXPIRE**:

  - Effect: When configured, if the user's password expires, the system prompts "the password has expired" upon login.

  - Client adaptation:

    - OGSQL client: When a user with an expired password logs in, an interactive password change window pops up, forcing the user to change the password. If the password change succeeds, the database connection is established; if it fails, the window is closed.

    - Other clients: directly report an error and exit the login process.

- **ACCOUNT {LOCK | UNLOCK}**: Effect: After an account is manually locked, the message "the account is locked" is displayed when the user attempts to log in, and login is prohibited; after unlocking, normal operation is restored.

- **PROFILE profile_name**: Profile configuration.

  > **Note:** The `PROFILE` referenced when creating a user must be created and configured in advance. If not explicitly specified, the default `PROFILE` is referenced by default. You can also explicitly reference the "DEFAULT" `PROFILE` by using double quotation marks.

  > **Note:** The system administrator `SYS` user references the default `PROFILE` by default. To prevent the security policy configuration of ordinary users from affecting the `SYS` user, it is advised to create a dedicated `PROFILE` for ordinary users.

- **ENCRYPTED**: Ciphertext password identifier.

  - Purpose: Indicates that the specified password is an encrypted ciphertext (recommended to be enclosed in single quotation marks). Password specifications are not validated when a ciphertext is specified.

  - Usage rules:

    - The `ENCRYPTED` keyword must be placed at the first parameter position after password; otherwise, an error is reported.

    - When a user is created in this manner, it is not possible to verify whether the plaintext password corresponding to the ciphertext complies with the `PASSWORD_MIN_LEN` parameter requirement in the `PROFILE`.

    - Users created in this way must still log in using plaintext passwords. This method is not recommended. Administrators can disable this creation method by setting the `enable_password_cipher` parameter.

- **PERMANENT**: Indicates a permanent user.

  - **Core Rule**: A user marked as `PERMANENT` is a non-deletable account. Only the `SYS` user can create or delete such accounts.

  - **Operation Restrictions**: After the account is created, only the `SYS` user can perform the following operations: changing the password, toggling the lockout status, manually setting password expiration, changing the associated `PROFILE`, and changing the user tablespace.

  - **Other Characteristics**: Except for the restricted operations described above, other characteristics of such accounts are consistent with those of ordinary accounts (for example, modifying the associated `PROFILE` of such an account has the same effect as for an ordinary account).

- **Special Characters**:

  |ID|Character|
  |---|---|
  |1|`|
  |2|~|
  |3|!|
  |4|@|
  |5|#|
  |6|$|
  |7|%|
  |8|^|
  |9|&|
  |10|*|
  |11|(|
  |12|)|
  |13|-|
  |14|_|
  |15|=|
  |16|+|
  |17|\|
  |18||
  |19|[|
  |20|{|
  |21|}|
  |22|]|
  |23|:|
  |24|'|
  |25|"|
  |26|,|
  |27|<|
  |28|.|
  |29|>|
  |30|/|
  |31|?|

## Examples

Create user mike with manual password expiration, prompting the user to change the password upon next login.

CREATE USER mike IDENTIFIED BY "123ggFGgh" PASSWORD EXPIRE;
