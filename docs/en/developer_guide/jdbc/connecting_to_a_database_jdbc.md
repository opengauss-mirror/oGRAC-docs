# Connecting to the Database

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-17T06:08:18.172Z pushedAt=2026-07-17T06:18:00.586Z -->

After a database connection is created, you can use it to execute SQL statements.

## Function Prototypes<a name="zh-cn_topic_0283137601_zh-cn_topic_0237120381_zh-cn_topic_0213179126_zh-cn_topic_0189251768_zh-cn_topic_0059779354_s78385b36d75447f68647a4fa4e18e4eb"></a>

JDBC provides three methods for creating a database connection.

- DriverManager.getConnection(String url);

- DriverManager.getConnection(String url, Properties info);

- DriverManager.getConnection(String url, String user, String password);

## Example<a name="zh-cn_topic_0283137601_zh-cn_topic_0237120381_zh-cn_topic_0213179126_zh-cn_topic_0189251768_zh-cn_topic_0059779354_sa87cf707a76c493997989289921f9202"></a>

```java
// The following code encapsulates the database connection operation as an interface, allowing connection to the database via the user name and password.
public static Connection getConnect(String username, String passwd) {
    // Driver class.
    String driver = "org.opengauss.Driver";
    // Database connection descriptor.
    String sourceURL = "jdbc:oGRAC://127.0.0.1:8000";
    Connection conn = null;

    try {
        // Load the driver.
        Class.forName(driver);
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }

    try {
        // Create a connection.
        conn = DriverManager.getConnection(sourceURL, username, passwd);
        System.out.println("Connection succeed!");
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }

    return conn;
}

// The following code uses a Properties object as a parameter to establish a connection.
public static Connection getConnectUseProp(String username, String passwd) {
    // Driver class.
    String driver = "org.opengauss.Driver";
    // Database connection descriptor.
    String sourceURL = "jdbc:oGRAC://127.0.0.1:8000";
    Connection conn = null;
    Properties info = new Properties();

    try {
        // Load the driver.
        Class.forName(driver);
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }

    try {
        info.setProperty("user", username);
        info.setProperty("password", passwd);
        // Create a connection.
        conn = DriverManager.getConnection(sourceURL, info);
        System.out.println("Connection succeed!");
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }

    return conn;
}
```
