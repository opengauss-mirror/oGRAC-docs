# ALTER SYSTEM KILL SESSION

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:04:02.788Z pushedAt=2026-08-10T08:40:37.058Z -->

## Function Description

Terminates a specified database session.

## Precautions

- **Permission requirements**: The `ALTER SYSTEM` system permission is required to perform this operation.

- **Restrictions**:

  - The current logged-in session cannot be terminated.

  - The core sessions reserved by the system cannot be terminated.

## Syntax

```sql
ALTER SYSTEM KILL SESSION 'session_id,serial#';
```

## Parameter Description

- `session_id`: indicates the session identifier (SID).

- `serial#`: indicates the session serial number.

## Examples

Preparations:

```
-- Delete the user (if exists).
DROP USER DDMUSER CASCADE;

-- Create a user.
CREATE USER DDMUSER IDENTIFIED BY password;

-- Grant basic permissions.
GRANT CONNECT, RESOURCE TO DDMUSER;

-- Grant the session viewing permission.
GRANT SELECT ON DV_SESSIONS TO DDMUSER;

-- Grant the session termination permission.
GRANT ALTER SYSTEM TO DDMUSER;
```

Connect to the database process:

```
-- Connect to the database using DDMUSER.
conn DDMUSER/password@127.0.0.1:1611

-- Query the session information of the DDMUSER user.
SELECT SID, SPID, SERIAL#, USERNAME, CLIENT_IP FROM DV_SESSIONS WHERE USERNAME = 'DDMUSER';
SID          SPID        SERIAL#      USERNAME                                                         CLIENT_IP                                                       
------------ ----------- ------------ ---------------------------------------------------------------- ----------------------------------------------------------------
130          639782      185          DDMUSER                                                          127.0.0.1                                                       
174          637288      184          DDMUSER                                                          127.0.0.1                                                       

2 rows fetched.
```

Terminate a specified non-current session:

```
ALTER system kill session '174,184';
```

Any attempt to terminate the current session will result in failure, accompanied by the error message: `The current session cannot be killed`.