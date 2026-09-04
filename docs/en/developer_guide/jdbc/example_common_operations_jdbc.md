# Common Operations

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-17T08:10:03.333Z pushedAt=2026-07-20T08:45:49.108Z -->

## Example 1<a name="zh-cn_topic_0283136826_zh-cn_topic_0237120386_zh-cn_topic_0213179152_zh-cn_topic_0189251739_section8820452474"></a>

This example demonstrates how to develop an application based on the JDBC interface provided by oGRAC.

```java
// DBtest.java
// The following demonstrates the main steps of JDBC-based development, including creating a database, creating a table, and inserting data.

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.CallableStatement;
import java.sql.Types;

public class DBTest {

  // Create a database connection.
  public static Connection GetConnection(String username, String passwd) {
    String driver = "org.opengauss.Driver";
    String sourceURL = "jdbc:oGRAC://localhost:8000";
    Connection conn = null;
    try {
      // Load the database driver.
      Class.forName(driver).newInstance();
    } catch (Exception e) {
      e.printStackTrace();
      return null;
    }

    try {
      // Create a database connection.
      conn = DriverManager.getConnection(sourceURL, username, passwd);
      System.out.println("Connection succeed!");
    } catch (Exception e) {
      e.printStackTrace();
      return null;
    }

    return conn;
  }

  // Execute a regular SQL statement to create the customer_t1 table.
  public static void CreateTable(Connection conn) {
    Statement stmt = null;
    try {
      stmt = conn.createStatement();

      // Execute a regular SQL statement.
      int rc = stmt
          .executeUpdate("CREATE TABLE customer_t1(c_customer_sk INTEGER, c_customer_name VARCHAR(32));");

      stmt.close();
    } catch (SQLException e) {
      if (stmt != null) {
        try {
          stmt.close();
        } catch (SQLException e1) {
          e1.printStackTrace();
        }
      }
      e.printStackTrace();
    }
  }

  // Execute a prepared statement to batch insert data.
  public static void BatchInsertData(Connection conn) {
    PreparedStatement pst = null;

    try {
      // Generate the prepared statement.
      pst = conn.prepareStatement("INSERT INTO customer_t1 VALUES (?,?)");
      for (int i = 0; i < 3; i++) {
        // Add parameters.
        pst.setInt(1, i);
        pst.setString(2, "data " + i);
        pst.addBatch();
      }
      // Execute the batch.
      pst.executeBatch();
      pst.close();
    } catch (SQLException e) {
      if (pst != null) {
        try {
          pst.close();
        } catch (SQLException e1) {
          e1.printStackTrace();
        }
      }
      e.printStackTrace();
    }
  }

  // Execute the prepared statement and update data.
  public static void ExecPreparedSQL(Connection conn) {
    PreparedStatement pstmt = null;
    try {
      pstmt = conn.prepareStatement(
          "UPDATE customer_t1 SET c_customer_name = ? WHERE c_customer_sk = 1");
      pstmt.setString(1, "new Data");
      int rowcount = pstmt.executeUpdate();
      pstmt.close();
    } catch (SQLException e) {
      if (pstmt != null) {
        try {
          pstmt.close();
        } catch (SQLException e1) {
          e1.printStackTrace();
        }
      }
      e.printStackTrace();
    }
  }


  // Execute the stored procedure.
  public static void ExecCallableSQL(Connection conn) {
    CallableStatement cstmt = null;
    try {
      // Create the stored procedure TESTPROC in advance.
      cstmt = conn.prepareCall("{? = CALL TESTPROC(?,?,?)}");
      cstmt.setInt(2, 50);
      cstmt.setInt(1, 20);
      cstmt.setInt(3, 90);
      cstmt.registerOutParameter(4, Types.INTEGER);  // Register the out parameter of integer type.
      cstmt.execute();
      int out = cstmt.getInt(4);  // Obtain the out parameter.
      System.out.println("The CallableStatement TESTPROC returns:" + out);
      cstmt.close();
    } catch (SQLException e) {
      if (cstmt != null) {
        try {
          cstmt.close();
        } catch (SQLException e1) {
          e1.printStackTrace();
        }
      }
      e.printStackTrace();
    }
  }
  

  /**
   * Main program, which calls each static method step by step.
   * @param args
   */
  public static void main(String[] args) {
    // Create a database connection.
    Connection conn = GetConnection("tester", "Password1234");

    // Create a table.
    CreateTable(conn);

    // Batch insert data.
    BatchInsertData(conn);

    // Execute the prepared statement and update data.
    ExecPreparedSQL(conn);

    // Execute the stored procedure.
    ExecCallableSQL(conn);

    // Close the database connection.
    try {
      conn.close();
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
}
```

## Example 2: Resolving Excessive Client Memory Usage<a name="zh-cn_topic_0283136826_zh-cn_topic_0237120386_zh-cn_topic_0213179152_zh-cn_topic_0189251739_section112115111092"></a>

This example primarily uses setFetchSize to adjust client memory usage. The underlying principle is to batch fetch server-side data through a database cursor. However, this approach increases network interactions and may result in some performance loss.

```java
Statement st = conn.createStatement();

// Open the cursor and fetch 50 rows of data at a time.
st.setFetchSize(50);
ResultSet rs = st.executeQuery("SELECT * FROM mytable");
while (rs.next()) {
    System.out.print("50 rows were returned.");
}
rs.close();

// Close the server cursor.
st.setFetchSize(0);
rs = st.executeQuery("SELECT * FROM mytable");
while (rs.next()) {
    System.out.print("all rows were returned.");
}
rs.close();

// Close the statement.
st.close();
conn.close();
```
