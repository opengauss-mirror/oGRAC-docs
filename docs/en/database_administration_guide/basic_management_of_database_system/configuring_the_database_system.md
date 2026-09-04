# Configuring the Database System

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T10:59:53.963Z pushedAt=2026-08-19T06:25:37.886Z -->

The oGRAC database provides a variety of system parameters that users can query or modify, allowing flexible adaptation to different environments and business scenarios.

## Viewing Database System Parameters

Through the system view DV_PARAMETERS, you can view the current database parameter configuration. The following explains each parameter:

- `name`: parameter name

- `value`: parameter value

- `RUNTIME_VALUE`: the value actually used at runtime

- `DEFAULT_VALUE`: default value of the parameter

- `ISDEFAULT`: whether the current value is the default value

- `MODIFIABLE`: whether the parameter can be modified

- `DESCRIPTION`: description of the parameter

- `RANGE`: value range of the parameter

- `DATATYPE`: data type of the parameter

- `EFFECTIVE`: how the parameter takes effect

    - `reboot`: takes effect only after a restart

    - `immediately`: takes effect dynamically

    - `reconnect`: takes effect after reconnection

## Configuring Database System Parameters

You can modify database system parameters using the ALTER SYSTEM statement.

The syntax for configuring parameters is as follows:

```sql
ALTER SYSTEM SET parameter_name = parameter_value [SCOPE = {PFILE | MEMORY | BOTH}];

--Example
ALTER SYSTEM SET SHARED_POOL_SIZE = 2G SCOPE = MEMORY;

SHOW PARAMETER SHARED_POOL_SIZE;
```

The meaning of each `SCOPE` parameter is as follows:

- `SCOPE=SPFILE`: The modification is written only to the server parameter file and takes effect after a restart.

- `SCOPE=MEMORY`: The modification affects only the current instance and takes effect immediately, but becomes invalid after a restart.

- `SCOPE=BOTH`: Both the memory and the parameter file are modified. The modification takes effect immediately and remains valid after a restart.

If necessary, the user must have the ALTER SYSTEM system permission.
