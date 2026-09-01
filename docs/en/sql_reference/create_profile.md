# CREATE PROFILE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:14:33.287Z pushedAt=2026-08-13T10:47:14.565Z -->

## Function Description

Creates a user profile, which is used to manage resource limits and password policies for database users. A profile can be shared by multiple users, facilitating unified management of user security and resource usage policies.

## Precautions

Creating or replacing a profile requires the corresponding system permissions. Improper parameter settings may cause account lockout or overly strict password policies. Configure the parameters appropriately based on actual service requirements.

## Syntax

```sql
CREATE PROFILE profile_name [REPLACE] LIMIT
{
    parameter1 [ { UNLIMITED | DEFAULT | value } ]
    [ parameter2 [ { UNLIMITED | DEFAULT | value } ] ]
    ...
}
```

## Parameter Description

### Basic Parameters

| Name | Description |
| ------ | ---- |
| profile_name | Name of the profile |
| REPLACE | Optional. If the specified profile already exists, replaces it. |

### Parameter Value Options

| Option | Description |
| ---- | ---- |
| UNLIMITED | Indicates no limit. |
| DEFAULT | Uses the system default value. |
| value | A specific value |

### Configurable Parameter List

| Name | Type | Default Value | Unit | Description |
| ------ | ---- | ------ | ---- | ---- |
| FAILED_LOGIN_ATTEMPTS | Integer | 10 | Count | Specifies the maximum number of failed login attempts for a user. The account will be locked after this number is exceeded. |
| PASSWORD_LIFE_TIME | Integer | 15552000 | Seconds | Specifies the validity period of a password. The password will expire after this period, and the user must change the password to log in. |
| PASSWORD_REUSE_TIME | Integer | UNLIMITED | Seconds | Specifies the number of days that must elapse before a password can be reused. If this parameter is set to an integer, `PASSWORD_REUSE_MAX` must be set to `UNLIMITED`. |
| PASSWORD_REUSE_MAX | Integer | UNLIMITED | Count | Specifies the number of password changes required before a password can be reused. If this parameter is set to an integer, `PASSWORD_REUSE_TIME` must be set to `UNLIMITED`. |
| PASSWORD_LOCK_TIME | Integer | 86400 | Seconds | Specifies the duration for which an account is locked. |
| PASSWORD_GRACE_TIME | Integer | 604800 | Seconds | Specifies the grace period after a password expires. During this period, a warning is displayed upon login, but the user can still log in. |
| SESSIONS_PER_USER | Integer | UNLIMITED | Count | Specifies the maximum number of concurrent sessions allowed per user. |
| PASSWORD_MIN_LEN | Integer | 8 | Characters | Specifies the minimum length of a password. |

## Examples

### Creating a Basic Profile

```
CREATE PROFILE app_user_profile LIMIT
    FAILED_LOGIN_ATTEMPTS 5;
```

### Replace an Existing Profile

```
CREATE OR REPLACE PROFILE app_user_profile LIMIT
    FAILED_LOGIN_ATTEMPTS 20;
```

### Creating a Profile Using Default Values

```
CREATE PROFILE default_profile LIMIT
    FAILED_LOGIN_ATTEMPTS DEFAULT;
```

### Creating an Unlimited Profile

```
CREATE PROFILE unlimited_profile LIMIT
    PASSWORD_LIFE_TIME UNLIMITED;
```
