# Closing a Connection

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-17T08:06:30.485Z pushedAt=2026-07-17T08:07:52.753Z -->

After completing data operations using a database connection, the database connection needs to be closed.

To close a database connection, you can directly call its `close` method. For example:

```java
Connection conn = DriverManager.getConnection("url","user","password");
conn.close();
```
