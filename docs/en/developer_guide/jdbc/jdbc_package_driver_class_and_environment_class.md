# JDBC Packages, Driver Classes, and Environment Classes<a name="ZH-CN_TOPIC_0289900723"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-16T10:53:36.420Z pushedAt=2026-07-16T11:26:58.767Z -->

Java Database Connectivity (JDBC) is a Java API for executing SQL statements. It provides a unified access interface for various relational databases, allowing applications to operate data based on it. oGRAC supports JDBC 4.0 features and requires JDK 1.8 to compile program code. The JDBC‑ODBC bridge is not supported.

## JDBC Packages<a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_zh-cn_topic_0189249669_zh-cn_topic_0059778950_s9694406852de4db0afcc99b19079f446"></a>

oGRAC provides two JDBC jar packages: `postgresql.jar` and `openGauss-jdbc-x.x.x.jar`. They have the same functionality and exist solely to resolve JDBC driver package name conflicts with PostgreSQL.

Execute `build.sh` in the Linux server-side source code directory to obtain the driver JAR packages `postgresql.jar` and `opengauss-jdbc-x.x.x.jar`. The packages are located in the `output` directory within the source code. They can also be obtained from the official release package, named `oGRAC-x.x.x-JDBC.tar.gz`.

- `postgresql.jar`: The main class name is `org.postgresql.Driver`, and the database connection URL prefix is `jdbc:oGRAC`.

- `opengauss-jdbc-x.x.x.jar`: The main class name is `org.opengauss.Driver`, and the database connection URL prefix is `jdbc:oGRAC`.

## Environment Classes<a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_section20655192913405"></a>

JDK 1.8 is required on the client. The configuration procedure is as follows:

1. Enter `java -version` in a DOS window to check whether the JDK version is JDK 1.8. If JDK is not installed, download the installation package from the official website and install it.

2. Configure the system environment variables by following the steps below.

    1. Right-click `This PC` and select `Properties`.

    2. On the `System` page, click "Advanced System Settings" in the left navigation pane.

    3. On the `System Properties` page, click `Environment Variables` on the `Advanced` tab page.

    4. On the `Environment Variables` page, click `New` or `Edit` in the `System variables` area to configure system variables. For variable descriptions, see [Table 1](#zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_table1625616152473).

        **Table 1**  Variables

        <a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_table1625616152473"></a>

        <table><thead align="left"><tr id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_row1525719152472"><th class="cellrowborder" valign="top" width="15.701570157015702%" id="mcps1.2.4.1.1"><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p52573155477"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p52573155477"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p52573155477"></a>Variable Name</p>
        </th>
        <th class="cellrowborder" valign="top" width="32.753275327532755%" id="mcps1.2.4.1.2"><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p525713159478"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p525713159478"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p525713159478"></a>Operation</p>
        </th>
        <th class="cellrowborder" valign="top" width="51.54515451545154%" id="mcps1.2.4.1.3"><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p8257141544718"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p8257141544718"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p8257141544718"></a>Variable Value</p>
        </th>
        </tr>
        </thead>
        <tbody><tr id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_row925712153479"><td class="cellrowborder" valign="top" width="15.701570157015702%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p182575153474"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p182575153474"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p182575153474"></a>JAVA_HOME</p>
        </td>
        <td class="cellrowborder" valign="top" width="32.753275327532755%" headers="mcps1.2.4.1.2 "><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_ul16913207507"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_ul16913207507"></a><ul id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_ul16913207507"><li>If it exists, click Edit.</li><li>If it does not exist, click New.</li></ul>
        </td>
        <td class="cellrowborder" valign="top" width="51.54515451545154%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p0217595132"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p0217595132"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p0217595132"></a>JAVA installation directory.</p>
        <p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1981211652"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1981211652"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1981211652"></a>Example: C:\Program Files\Java\jdk1.8.0_131</p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_row5257111564711"><td class="cellrowborder" valign="top" width="15.701570157015702%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1325721584719"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1325721584719"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1325721584719"></a>Path</p>
        </td>
        <td class="cellrowborder" valign="top" width="32.753275327532755%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p7257191511477"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p7257191511477"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p7257191511477"></a>Edit</p>
        </td>
        <td class="cellrowborder" valign="top" width="51.54515451545154%" headers="mcps1.2.4.1.3 "><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_ul108931343135117"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_ul108931343135117"></a><ul id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_ul108931343135117"><li>If JAVA_HOME is configured, prepend the following to the variable value: %JAVA_HOME%\bin;</li><li>If JAVA_HOME is not configured, prepend the full JAVA installation path to the variable value:<p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p54286393517"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p54286393517"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p54286393517"></a>C:\Program Files\Java\jdk1.8.0_131\bin;</p>
        </li></ul>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_row325881510471"><td class="cellrowborder" valign="top" width="15.701570157015702%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1625812150476"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1625812150476"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1625812150476"></a>CLASSPATH</p>
        </td>
        <td class="cellrowborder" valign="top" width="32.753275327532755%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p13527826155219"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p13527826155219"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p13527826155219"></a>New</p></td>
        <td class="cellrowborder" valign="top" width="51.54515451545154%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1325841534712"><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1325841534712"></a><a name="zh-cn_topic_0283136979_zh-cn_topic_0237120378_zh-cn_topic_0213179123_p1325841534712"></a>.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar;</p></td>
        </tr>
        </tbody>
        </table>
