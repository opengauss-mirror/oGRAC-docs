# Loading the Driver

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-16T11:33:26.071Z pushedAt=2026-07-17T06:06:44.476Z -->

Before creating a database connection, you need to load the database driver first.

Method for loading the JDBC driver:

- If you are using the `postgresql.jar` package, pass the parameter at JVM startup: `java -Djdbc.drivers=org.postgresql.Driver jdbctest`

- If you are using the `openGauss-jdbc-x.x.x.jar` package, pass the parameter at JVM startup: `java -Djdbc.drivers=org.opengauss.Driver jdbctest`

  > [!NOTE]
  > `jdbctest` is the test program name.
  