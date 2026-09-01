# DROP PROFILE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-28T10:36:45.244Z pushedAt=2026-07-29T06:14:15.848Z -->

## Description

The `DROP PROFILE` statement is used to drop an existing configuration file (profile) in the database. A profile is a collection of settings used to manage resource limits and password policies for database users.

## Notes

The user executing the `DROP PROFILE` statement must have the `DROP PROFILE` system privilege. `DEFAULT PROFILE` is the system default profile and cannot be dropped. If a profile is being used by other users, the `CASCADE` option must be used to drop it after resetting their profiles to the default profile.

## Syntax

```sql
DROP PROFILE profile_name [CASCADE];
```

## Parameter Description

- **profile_name**: The name of the profile to be dropped.

- **CASCADE**: An optional parameter. If a profile is currently in use by users, specifying this option resets the profiles of those users to the default profile and then drops the profile. If this option is not specified and the profile is in use, the drop operation will fail.

## Examples

### Example 1: Dropping an unused profile

```
DROP PROFILE APP_USER_PROFILE;
```

### Example 2: Using `CASCADE` to drop a profile that is in use

```
DROP PROFILE APP_USER_PROFILE CASCADE;
```
