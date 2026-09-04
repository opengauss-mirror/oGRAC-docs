# View Management

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:01:09.177Z pushedAt=2026-08-20T08:49:38.763Z -->

A view (`View`) is a virtual table built from the query results of one or more tables (or other views). It does not store actual data; instead, it dynamically executes the `SQL` statement defined for it at query time to return data. Users can perform `SELECT` operations on a view just as they would on a regular table, but cannot directly insert, update, or delete data. The data carried by a view is closely related to the base table data at the time the view was built; if the data in the base tables changes, the data queried from the view changes accordingly.

Users can create a view using `CREATE VIEW VIEW_NAME AS` or `CREATE OR REPLACE VIEW VIEW_NAME AS`, with a `SELECT` statement following `AS`.

```sql
CREATE OR REPLACE VIEW USER_V1 AS SELECT ID, AGE, NAME FROM USERS WHERE AGE > 18;
```

Users can query view-related information through `DB_VIEWS` or `MY_VIEWS`, as well as the `DESC` command.

```sql
SQL> SELECT * FROM DB_VIEWS WHERE VIEW_NAME = 'USER_V1';

OWNER                                                            VIEW_NAME                                                        VIEW_TYPE COLUMN_COUNT TEXT                                                             TEXT_LENGTH  CREATED_TIME           LAST_DDL_TIME         
---------------------------------------------------------------- ---------------------------------------------------------------- --------- ------------ ---------------------------------------------------------------- ------------ ---------------------- ----------------------
SYS                                                              USER_V1                                                          NORMAL    3            SELECT ID, AGE, NAME FROM USERS WHERE AGE > 18                   46           2025-11-13 17:16:37    2025-11-13 17:16:37   

1 rows fetched.

SQL> desc USER_V1

Name                                Null?    Type                                
----------------------------------- -------- ------------------------------------
ID                                  NOT NULL BINARY_INTEGER                      
AGE                                          BINARY_INTEGER                      
NAME                                         VARCHAR(10 BYTE)                    
```

Querying a view works the same as querying a table. Views can be removed with `DROP VIEW`.

```sql
DROP VIEW USER_V1;
DROP VIEW IF EXISTS USER_V1;
```
