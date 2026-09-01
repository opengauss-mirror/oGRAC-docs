# Release Notes

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:49:34.125Z pushedAt=2026-08-17T00:46:29.694Z -->

## Version Introduction

openGauss Real Application Cluster (oGRAC) 7.0.0-RC3 is a multi-primary database innovation version released by the openGauss community in March 2026, with a lifecycle of 0.5 years. This version is the first release of oGRAC.

Key features:

- Multi-read multi-write support

    Provides a kernel designed for easy scaling into a multi-primary architecture, ensuring transaction and page cache consistency across multiple nodes based on Distributed Memory Service (DMS). All instances share the same storage and support concurrent read and write.

- Standard SQL support

    Supports standard SQL92/SQL99/SQL2003 specifications, GBK and UTF-8 character sets, SQL standard functions and analytic functions, and stored procedures.

- Database storage management

    Supports tablespaces, allowing different tables to be placed in different storage locations.

- App programming interface

    Supports standard JDBC and ODBC drivers.

### Resolved Issues

For the complete list of issues, see [Issues](https://gitcode.com/opengauss/oGRAC/issues).

For the complete kernel commit history, see [Commit History](https://gitcode.com/opengauss/oGRAC/tree/7.0.0-RC3).

See the table below for the resolved issues.

| Issue                                                        | Associated Repository                                | Description                                                     |
| ------------------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| [121](https://gitcode.com/opengauss/oGRAC/issues/121) | oGRAC | [Bug]: Core dump when creating a table with arrays |
| [119](https://gitcode.com/opengauss/oGRAC/issues/119) | oGRAC | [Bug]: Core dump caused by non-correlated query with EXISTS condition |
| [107](https://gitcode.com/opengauss/oGRAC/issues/107) | oGRAC | [Bug]: Core dump caused by WITH AS query |
| [103](https://gitcode.com/opengauss/oGRAC/issues/103) | oGRAC | [Bug]: Fixed null pointer core dump |
| [94](https://gitcode.com/opengauss/oGRAC/issues/94)  | oGRAC  | [Bug]: SQL engine core dump during SELECT |
| [87](https://gitcode.com/opengauss/oGRAC/issues/87)  | oGRAC  | [Bug]: Planner core dump during SELECT |
| [71](https://gitcode.com/opengauss/oGRAC/issues/71)  | oGRAC  | [Bug]: Core dump with GROUP_CONCAT + ORDER BY query |
| [29](https://gitcode.com/opengauss/oGRAC/issues/29)  | oGRAC  | [Bug]: Core dump when executing GROUP_CONCAT test case |
| [26](https://gitcode.com/opengauss/oGRAC/issues/26)  | oGRAC  | [Bug]: Core dump when executing tc_group_concat_bound_020 |

### Inherited Features

  - Basic features: [Basic SQL Capabilities](../about_ograc/product_description/sql_ability.md#basic-sql-capabilities), [SQL Optimizer Capabilities](../about_ograc/product_description/sql_ability.md#sql-optimizer-capabilities), [Transaction Management](../about_ograc/product_architecture/transaction_mechanism.md#transaction-management), etc.
  - [Multi-Write Capability Overview](../about_ograc/product_architecture/technical_feature_overview_of_database_multi_write_capability.md#1-multi-write-capability-overview)
  - Storage engine features
    - [CKPT](../about_ograc/product_architecture/ckpt.md), which establishes a synchronization point between memory and disk to ensure data consistency and durability, and accelerates the database recovery process.
    - [Online DDL](../about_ograc/product_architecture/online_ddl.md), which completes table definition modifications without affecting or minimally affecting the normal service of the database.
    - [Redo](../about_ograc/product_architecture/redo.md), which ensures transaction durability, meaning that once a transaction is committed, the modifications it made to data will never be lost, even if a system crash occurs subsequently.
  - Maintainability
    - [WSR Report](../about_ograc/maintainability.md#wsr-report), which generates a performance analysis report for a period by taking database snapshots and using the statistical data collected from snapshots generated at two different points in time.
    - [Statistical Views](../about_ograc/maintainability.md#statistical-views), which provide multiple views for users to view system information. By functional dimension, they can be divided into DBA views, user views, and performance views. Combining the information from these views allows observation of the current internal running status of the database.
  - App driver
    - [JDBC Driver](../developer_guide/jdbc/jdbc_package_driver_class_and_environment_class.md#jdbc-packages-driver-classes-and-environment-classes). Java Database Connectivity (JDBC) is a Java API for executing SQL statements, providing a unified access interface for various relational databases, on which apps can operate data.
    - [ODBC Driver](../developer_guide/odbc/linux_configure_data_source.md#configuring-a-data-source-on-linux). Open Database Connectivity (ODBC) is a widely accepted API for database access. It is based on the Call-Level Interface (CLI) specifications for database APIs from the Open Group and ISO/IEC, using Structured Query Language (SQL) as its database access language.

### CVE Vulnerabilities

This version is an innovation release published in March 2026. The CVEs involved in this version can be found at [CVE](https://opengauss.org/en/cve).

## Version Usage Notes

- For version technical specifications, refer to [Specifications](../about_ograc/specification/specification.md#table-specifications) in *About oGRAC*.
- Refer to [oGRAC Single-Node Local Installation](../installation_guide/single_node_guide/local_installation_on_a_single_node.md) or [oGRAC Two-Node Deployment](../installation_guide/two_nodes_guide/ograc_two_node_installation.md) in the *Installation Guide* to select the deployment architecture that meets your business needs, ensuring the reliability and availability of the database cluster.

## Source Code

oGRAC includes multiple code repositories, including patch code repositories for referenced open-source software, JDBC driver code repository, ODBC driver code repository, database server code repository, and documentation repository:

- Open-source software code repository: [https://gitcode.com/opengauss/openGauss-third\_party](https://gitcode.com/opengauss/openGauss-third_party)
- JDBC driver code repository: [https://gitcode.com/opengauss/openGauss-connector-jdbc](https://gitcode.com/opengauss/openGauss-connector-jdbc)
- ODBC driver code repository: [https://gitcode.com/opengauss/openGauss-connector-odbc](https://gitcode.com/opengauss/openGauss-connector-odbc)
- Database server code repository: [https://gitcode.com/opengauss/oGRAC](https://gitcode.com/opengauss/oGRAC)
- Database documentation repository: [https://gitcode.com/opengauss/docs](https://gitcode.com/opengauss/docs)

## Notes for Users

oGRAC is an open-source database that supports multi-read multi-write operations. It is licensed under the Mulan Permissive Software License (Mulan PSL V2), allowing users to freely copy, use, modify, and distribute the software, whether modified or not.

The version number of oGRAC follows the *X.Y.0-RCx* format, designed to distinguish different types of version updates. Specifically:

 + LTS release (*X.0.0*): Released every two years, marking a version with long-term stable support, suitable for large-scale deployment and production environments. The community commits to providing three years of maintenance support for each LTS release, ensuring its continued stability and security.
 + Innovation release (*X.Y.0-RCx*): Released every six months, aimed at rapid iteration and providing the latest features and technology previews. These versions are primarily intended for user testing and innovation collaboration, with the community offering six months of maintenance support to encourage exploration and feedback.
 + Patch release (*X.Y.0*): Released as needed when critical issues arise, aimed at quickly fixing critical bugs to ensure stable system operation.

## Acknowledgments

We sincerely thank all developers and partners who participated in and contributed to the release of oGRAC 7.0.0-RC3. It is your hard work that has enabled the smooth release of this version and paved the way for the continued growth of oGRAC.
