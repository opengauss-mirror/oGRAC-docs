# Accessing and Using oGRAC

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:48:12.737Z pushedAt=2026-08-17T00:46:29.676Z -->

## Accessing oGRAC

Client tools for connecting to the database include ogsql and app interfaces (such as JDBC).

- ogsql is the client tool bundled with oGRAC. Using ogsql to connect to the database, you can interactively enter, edit, and execute SQL statements. For details, see [ogsql Usage Instructions](../tool_and_commandreference/client_tool/ogsql_instructions.md).
- Users can use standard database app interfaces (such as JDBC) to develop oGRAC-based apps. For details, see [JDBC Guide](../developer_guide/jdbc/development_process_jdbc.md).

---

## Using oGRAC

The following describes the basic operations of using oGRAC. Through the following content, you can learn how to create a database, create tables, insert data into tables, and query data in tables.

### Prerequisites

You have completed the installation of oGRAC.

### Procedure

1. Log in to the primary database node as the installation user.

2. Connect to the database.

    After the database installation is complete, the database starts by default. You can connect to this database for the first connection.

    Run the following command to connect to the database:

    ```shell
    ogsql / as sysdba
    ```

    Alternatively, run the following command to connect to the database:

    ```shell
    ogsql user_name/user_password@127.0.0.1:port
    ```

    A successful connection brings up the interactive interface..

3. Create a database user.

    Run the following command to create a database user:

    ```sql
    create user user_name identified by 'user_password';
    ```

    If the following information is displayed, the creation is successful:

    ```shell
    Succeed.
    ```

    Run the following command to set the user as a system administrator:

    ```sql
    grant all to user_name;
    ```

4. Create a table.

    Run the following command to create a table named `mytable` with only one column. The column name is `firstcol` and the column type is integer.

    ```sql
    create table mytable (firstcol int);
    ```

    The following output indicates that the table has been created successfully:

    ```shell
    Succeed.
    ```

    Run the following command to insert data into the table:

    ```sql
    insert into mytable values (1);
    ```

    If the following information is displayed, the insertion is successful:

    ```shell
    1 rows affected.
    ```

    Run the following command to query data in the table:

    ```sql
    select * from mytable;
    ```

    The query result is as follows:

    ```shell
    FIRSTCOL
    ------------
    1

    1 rows fetched.
    ```
