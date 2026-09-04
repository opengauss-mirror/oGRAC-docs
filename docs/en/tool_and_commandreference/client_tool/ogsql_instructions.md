# ogsql Usage Instructions

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:49:32.375Z pushedAt=2026-08-17T00:46:29.692Z -->

## Overview

ogsql is a SQL developer command-line tool provided by oGRAC, used for connecting to databases, executing SQL statements and scripts, and managing database objects. This tool supports both interactive and non-interactive usage modes.

## Command Line Syntax

### Basic Syntax

ogsql supports three main usage modes:

```bash
# Mode 1: Display help or version information.
ogsql -h
ogsql -v

# Mode 2: Enter interactive mode.
ogsql

# Mode 3: Specify login information and options to execute SQL statements in non-interactive mode.
ogsql [ <logon> [<options>] [<start>] ]
```

### Login Information Format

ogsql supports the following login formats:

```sh
[ user [ /password ] @{host:port}[,...] ] [as sysdba]
```

As well as the special administrator login format:

```sh
/ as { sysdba } [ host:port ]
```

Parameter description:

- `user`: Login username.
- `password`: Login user password. If omitted, interactive mode will be used (recommended for security reasons).
- `host`: IP address of the database server, supporting both IPv4 and IPv6.
- `port`: Port number of the database server.
- `sysdba`: Database administrator identity.

### Options

ogsql supports the following options:

```sh
[-q] [-w <timeout>] [-a] [-D "data_home_path"]
```

Option description:

- `-q`: Disables SSL login authentication.
- `-w \<timeout>`: Timeout for the client to connect to the database, in seconds.
  - The default value is `60` seconds.
  - The special value `-1` indicates an infinite timeout.
  - The special value `0` indicates no waiting.
- `-a`: Prints the executed SQL statements.
  - Can be used together with `-f` to print and execute SQL statements in a SQL script file.
- `-D`: Specifies the data directory path.

### Startup Options

ogsql supports the following startup options (only one can be used at a time):

```sh
[-c "execute-sql-command"] | [-f "execute-sql-file"] | [-s "destination-file"]
```

Option description:

- `-c`: Executes the specified SQL statement.
- `-f`: Executes the specified SQL script file.
- `-s`: Redirects the command prompt and outputs to a specified file.

## Examples

### Basic Login

```bash
# Log in as an administrator without a password.
ogsql / as sysdba

# Log in as a specified user via the IP address and port.
ogsql user/user_pwd@127.0.0.1:1611
```

### Executing SQL Statements

```bash
# Execute a single SQL statement after login.
ogsql user/user_pwd@127.0.0.1:1611 -c "SELECT 1 FROM SYS_DUMMY"

# Execute a SQL script file.
ogsql user/user_pwd@127.0.0.1:1611 -f "/home/user/example.sql"

# Print and execute the statements in a SQL script file.
ogsql user/user_pwd@127.0.0.1:1611 -a -f "/home/user/example.sql"
```

### Setting the Connection Timeout

```bash
# Set the connection timeout to 30 seconds.
ogsql user/user_pwd@127.0.0.1:1611 -w 30

# Set the infinite timeout.
ogsql user/user_pwd@127.0.0.1:1611 -w -1
```

## Interactive Commands

In interactive mode, ogsql supports the following commands:

| Command | Description | Example |
|------|------|------|
| CONN | Connect to the database. | `CONN user/password@host:port` |
| EXIT/QUIT | Exit ogsql | `EXIT` |
| SHOW | Display system information. | `SHOW PARAMETERS` |
| DESC | Describe a database object. | `DESC table_name` |
| @file | Execute an SQL script file. | `@/home/user/example.sql` |
| @@file | Execute a nested SQL script file. | `@@/home/user/include.sql` |
| SPOOL | Save output to a file. | `SPOOL output.txt` |
| CLEAR | Clear the screen. | `CLEAR` |
| SET | Set ogsql environment variables. | `SET TIMING ON` |
| COLUMN | Set column display format. | `COLUMN column_name FORMAT A20` |
| WHENEVER | Set error handling. | `WHENEVER SQLERROR EXIT` |
| PROMPT | Display a prompt message. | `PROMPT 'Start executing the script'` |

## Parameter Binding

ogsql supports parameter binding in SQL statements, in the format of `:parameter_name`. When executing an SQL statement that contains parameters, ogsql prompts the user to enter the parameter values.

Example:

```sql
SELECT * FROM employees WHERE department_id = :dept_id;
```

During execution, ogsql prompts the user to enter the value of `dept_id`.

## Environment Variable Settings

Run the `SET` commands to configure the runtime environment of ogsql:

```sql
-- -- Enable execution time statistics.
SET TIMING ON;

-- -- Enable SQL statement display.
SET ECHO ON;
```

## Output Redirection

Run the `SPOOL` commands to redirect the output of ogsql to a file:

```sql
-- Redirect output to a file.
SPOOL output.txt;

-- Execute an SQL statement.
SELECT * FROM employees;

-- Stop output redirection.
SPOOL OFF;
```

## Error Handling

Run the `WHENEVER` commands to set the behavior of ogsql when an error is encountered:

```sql
-- Exit upon encountering an SQL error.
WHENEVER SQLERROR EXIT;

-- Continue execution upon encountering an SQL error but display the error.
WHENEVER SQLERROR CONTINUE;
```

## Metadata Query Examples

The following are some commonly used metadata query examples:

```sql
-- View all users.
SELECT * FROM ADM_USERS;

-- View all tables.
SELECT * FROM ADM_TABLES;

-- View the table structure.
DESC table_name;

-- View system parameters.
SHOW PARAMETERS;
```
