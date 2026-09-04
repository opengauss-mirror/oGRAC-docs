# ALTER USER

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:02.901Z pushedAt=2026-08-25T03:44:34.212Z -->

## Function Description

Modifies the attributes of **an existing database user**. Common use cases include but are not limited to:

- Modifying the login password

- Locking/Unlocking the user account

- Changing the profile assignment

- Setting the default tablespace

## Precautions

### Permission Requirements

- To modify attributes of **other users**, the `ALTER USER` system permission is required.

- To change only **your own password**, this permission is not required.

### Permission Scope

- **Common users (with `ALTER USER` permission)**

  - Can change passwords of users other than database administrators (DBAs) and the `SYS` user.

- **DBAs**

  - Can change passwords of all users except the `SYS` user.

- **`SYS` user**

  - Can change passwords of all users without restrictions.

### Other Constraints

- If the target user does not exist, an error is returned: `user <name> does not exist`.

- This statement cannot be executed **during the rollback phase of database recovery**.

- In audit logs, the password field in SQL statements is masked with asterisks (*) for security purposes.

## Syntax

```sql
ALTER USER user_name
{
  IDENTIFIED BY newPassword [REPLACE oldPassword]
| PASSWORD EXPIRE
| ACCOUNT { LOCK | UNLOCK }
| PROFILE profileName
| DEFAULT TABLESPACE tableSpaceName
}
[ , ... ];
```

## Parameter Description

### Username

- `user_name`:

  - Must be an existing user in the database.

### Password-Related Parameters

- `IDENTIFIED BY`:

  - Sets a new password for the user.

- `newPassword`:

  > **NOTE:** The password must comply with the following rules.
  >
  > - **Length requirements**: The length must be no shorter than the `PASSWORD_MIN_LEN` value of the associated profile.
  >     The maximum length is 64 characters.
  >
  > - **Profile synchronization**: If `PASSWORD_MIN_LEN` is modified at the same time, the newly set value takes precedence for this password validation.
  >
  > - **Character rules**: When the password is not enclosed in single quotation marks (''), the first character must be a letter, a number sign (#), or an underscore (_).
  >     The password must not be identical to the username or its reverse (case-insensitive).
  >
  > - **Complexity requirements**: The password must contain at least **3 of the following character types**:  
  >       - Digits  
  >       - Lowercase letters  
  >       - Uppercase letters  
  >       - Spaces or special characters
  >
  > - **Special character handling**: If the password contains spaces or special characters other than underscores (_), number signs (#), and dollar signs ($), it must be enclosed with single quotation marks ('').
  >
  > - **Password change requirement**: The new password must differ from the old password in at least **2 character positions**.
  >
  > - **Special notes for `ogsql`**: The `$` character in the password must be escaped with a backslash \(\\\).

- `REPLACE oldPassword`:

  - **If `REPLACE` is not specified**, the old password is not verified.

  - **If `REPLACE` is specified**, the correct old password must be provided.

  - **Parameter restriction**: If `REPLACE_PASSWORD_VERIFY` is `TRUE`, common users must use `REPLACE` to change their password.

### Account and Attribute Control

- `PASSWORD EXPIRE`:

  - Sets the user password to the expired state.

  - The user must change the password upon next login (as prompted by `ogsql`).

- `ACCOUNT LOCK`:

  - Locks the user account and prohibits login.

- `ACCOUNT UNLOCK`:

  - Unlocks the user account and restores login capability.

- `PROFILE profileName`:

  - Assigns an existing profile to the user.

- `DEFAULT TABLESPACE tableSpaceName`:

  - Sets the user's default tablespace.

### Supported Special Characters

- **Common symbols**: `` ` ~ ! @ # $ % ^ ``

- **Operators and connectors**: `` & * ( ) - _ = + ``

- **Separators and structural symbols**: `` [ ] { } | : ' " ``

- **Comparison and path symbols**: `` < > . , / ? ``

## Examples

### Create a User and Specify an Initial Password

```
CREATE USER userName IDENTIFIED BY oldPassword;
```

### Change the Password and Verify the Old Password

```
ALTER USER userName IDENTIFIED BY newPassword REPLACE oldPassword;
```

### Lock a User

```
ALTER USER userName ACCOUNT LOCK;
```

### Unlock a User

```
ALTER USER userName ACCOUNT UNLOCK;
```

### Set the Password as Expired

```
ALTER USER userName PASSWORD EXPIRE;
```