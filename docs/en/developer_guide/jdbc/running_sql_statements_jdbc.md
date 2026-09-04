# Executing SQL Statements<a name="ZH-CN_TOPIC_0289900186"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-17T06:21:17.374Z pushedAt=2026-07-17T06:58:35.653Z -->

## Executing Regular SQL Statements<a name="zh-cn_topic_0283137004_zh-cn_topic_0237120383_zh-cn_topic_0213179129_zh-cn_topic_0189250824_zh-cn_topic_0059777674_s6d6619f4f2df48198e8e7a32ccc4b47a"></a>

To manipulate database data by executing SQL statements (statements without parameter passing), an application must follow these steps:

1. Call the createStatement method of `Connection` to create a statement object.

    ```java
    Connection conn = DriverManager.getConnection("url","user","password");
    Statement stmt = conn.createStatement();
    ```

2. Call the executeUpdate method of `Statement` to execute the SQL statement.

    ```java
    int rc = stmt.executeUpdate("CREATE TABLE customer_t1(c_customer_sk INTEGER, c_customer_name VARCHAR(32));");
    ```

3. Close the statement object.

    ```java
    stmt.close();
    ```

## Executing Prepared SQL Statements<a name="zh-cn_topic_0283137004_zh-cn_topic_0237120383_zh-cn_topic_0213179129_zh-cn_topic_0189250824_zh-cn_topic_0059777674_scea08fc60d7c4db0ae5f31990a842a03"></a>

A prepared statement is compiled and optimized only once, and can then be reused multiple times with different parameter values. Since it is precompiled, subsequent executions take less time. Therefore, if a statement is to be executed multiple times, use a prepared statement.

1. Call the prepareStatement method of `Connection` to create a prepared statement object.

    ```java
    PreparedStatement pstmt = con.prepareStatement("UPDATE customer_t1 SET c_customer_name = ? WHERE c_customer_sk = 1");
    ```

2. Call the setShort method of `PreparedStatement` to set parameters.

    ```java
    pstmt.setShort(1, (short)2);
    ```

3. Call the executeUpdate method of `PreparedStatement` to execute the prepared SQL statement.

    ```java
    int rowcount = pstmt.executeUpdate();
    ```

4. Call the close method of `PreparedStatement` to close the prepared statement object.

    ```java
    pstmt.close();
    ```

## Calling Stored Procedures<a name="zh-cn_topic_0283137004_zh-cn_topic_0237120383_zh-cn_topic_0213179129_zh-cn_topic_0189250824_zh-cn_topic_0059777674_sed43ac42d9414b01bfad409279941f46"></a>

oGRAC supports directly calling pre-created stored procedures through JDBC. The steps are as follows:

1. Call the prepareCall method of `Connection` to create a call statement object.

    ```java
    Connection myConn = DriverManager.getConnection("url","user","password");
    CallableStatement cstmt = myConn.prepareCall("{? = CALL TESTPROC(?,?,?)}");
    ```

2. Call the setInt method of `CallableStatement` to set parameters.

    ```java
    cstmt.setInt(2, 50);
    cstmt.setInt(1, 20);
    cstmt.setInt(3, 90);
    ```

3. Call the registerOutParameter method of `CallableStatement` to register output parameters.

    ```java
    cstmt.registerOutParameter(4, Types.INTEGER);  // Register the OUT parameter of integer type.
    ```

4. Call the execute method of `CallableStatement` to execute the call.

    ```java
    cstmt.execute();
    ```

5. Call the getInt method of `CallableStatement` to obtain the output parameter.

    ```java
    int out = cstmt.getInt(4);  // Obtain the OUT parameter.
    ```

6. Call the close method of `CallableStatement` to close the call statement.

    ```java
    cstmt.close();
    ```

## Executing Batch Processing<a name="zh-cn_topic_0283137004_zh-cn_topic_0237120383_zh-cn_topic_0213179129_zh-cn_topic_0189250824_zh-cn_topic_0059777674_sb0c28cebb51d482c8bd996ce7fef3a6c"></a>

When processing multiple similar data entries with a single prepared statement, the database creates the execution plan only once, saving statement compilation and optimization time. The execution can be performed as follows:

1. Call the prepareStatement method of `Connection` to create a prepared statement object.

    ```java
    Connection conn = DriverManager.getConnection("url","user","password");
    PreparedStatement pstmt = conn.prepareStatement("INSERT INTO customer_t1 VALUES (?)");
    ```

2. For each data entry, call setShort to set the parameters, and call addBatch to add the entry to the parameter list.

    ```java
    pstmt.setShort(1, (short)2);
    pstmt.addBatch();
    ```

3. Call the executeBatch method of `PreparedStatement` to execute batch processing.

    ```java
    int[] rowcount = pstmt.executeBatch();
    ```

4. Call the close method of `PreparedStatement` to close the prepared statement object.

    ```java
    pstmt.close();
    ```

    > [!NOTE]
    > In actual batch processing, do not terminate the execution of the batch processing program, as this will degrade database performance. Therefore, when executing batch processing, you should disable the auto-commit feature and commit every few rows. The statement to disable auto-commit is: conn.setAutoCommit(false);
