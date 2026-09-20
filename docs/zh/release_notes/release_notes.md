# 发行说明

## 版本介绍

oGRAC（openGauss Real Application Cluster）7.0.0-LTS 是 openGauss 社区 2026 年 9 月发布的多主数据库LTS版本，该版本生命周期为 3 年。本版本为 oGRAC 的第一个长期稳定版本。

主要功能如下：

- 支持多读多写

    提供便于扩展成多主架构的内核，基于分布式内存服务（DMS）保障多节点间事务、页面缓存一致性，各实例共享同一份存储并支持并发读写。

- 标准SQL支持

    支持标准的 SQL92/SQL99/SQL2003 规范，支持 GBK、UTF-8 字符集，支持 SQL 标准函数与分析函数，支持存储过程。

- 数据库存储管理功能

    支持表空间，可以将不同表规划到不同的存储位置。

- 应用程序接口

    支持标准 JDBC、ODBC 驱动。

## 新增特性

此处说明的是oGRAC 7.0.0-LTS版本，在7.0.0-RC3版本功能的基础上，新增如下特性：

- **高性能**：
  - 结合线程绑核动态配置、表锁优化、undo page预分配等优化性能。[#300](https://atomgit.com/opengauss/oGRAC/pull/300) [@hwworkholic](https://atomgit.com/hwworkholic)

- **高可靠**：支持[RBPS(Recovery buffer pool server)](../tool_and_commandreference/server_tool/rbps_instructions.md)
  - 数据库可在恢复过程中使用 RBPS 中保存的页面，从而缩短节点故障恢复场景下的RTO时间。[#298](https://atomgit.com/opengauss/oGRAC/pull/298) [#337](https://atomgit.com/opengauss/oGRAC/pull/337) [@Miyano](https://atomgit.com/Miyano)

- **高可靠**：[备份恢复](../database_administration_guide/online_backup_offline_restore.md)支持离线操作
  - 全量备份及全量/增量恢复均支持离线操作。[#383](https://atomgit.com/opengauss/oGRAC/pull/383) [@gcw_Y2UmAK9J](https://atomgit.com/gcw_Y2UmAK9J)

- **企业级能力增强**：
  - [ogsql](../tool_and_commandreference/client_tool/ogsql_instructions.md#交互式输入编辑)支持readline，关键字自动补齐。[#316](https://atomgit.com/opengauss/oGRAC/pull/316) [@leweiyang](https://atomgit.com/leweiyang)
  - 支持查看[DSS设备信息](../database_administration_guide/basic_management_of_database_system/dynamic_views.md#dv_dss_time_stats)、[SQL语句历史执行计划及各计划执行信息](../database_administration_guide/basic_management_of_database_system/dynamic_views.md#dv_sql_execution)、[慢SQL信息](../database_administration_guide/basic_management_of_database_system/dynamic_views.md#dv_slow_sql)、[本地节点的缓存区页面信息](../database_administration_guide/basic_management_of_database_system/dynamic_views.md#dv_drc_buf_info)、[本地锁信息](../database_administration_guide/basic_management_of_database_system/dynamic_views.md#dv_drc_local_lock_info)、[本地节点缓冲区控制块信息](../database_administration_guide/basic_management_of_database_system/dynamic_views.md#dv_buf_ctrl_info)视图。[#93](https://atomgit.com/opengauss/oGRAC/pull/93) [@jyn88](https://atomgit.com/jyn88)

- **工具增强**:
  - 支持[Oracle到oGRAC的全量数据迁移工具](https://docs.opengauss.org/zh/docs/latest/data_migration_guide/oracle2ograc_migration.html)。提供了从 Oracle 到 oGRAC 全量数据和对象的迁移能力，全量数据迁移采用多表并行迁移，全量对象支持表、约束、索引、外键、视图、函数、触发器、存储过程和序列的迁移。 [#409](https://atomgit.com/opengauss/debezium/pull/409) [#411](https://atomgit.com/opengauss/debezium/pull/411) [#413](https://atomgit.com/opengauss/debezium/pull/413) [#414](https://atomgit.com/opengauss/debezium/pull/414) [#415](https://atomgit.com/opengauss/debezium/pull/415) [@wang4721](https://atomgit.com/wang4721)

## 版本兼容性说明

- 不涉及

### 已修复问题

完整问题清单请参见[完整问题清单](https://atomgit.com/opengauss/oGRAC/issues)。
完整的内核提交记录请参见[提交记录](https://atomgit.com/opengauss/oGRAC/tree/7.0.0)。

已修复问题请参见下表。

| ISSUE                                                        | 关联仓库                                | 问题描述                                                     |
| ------------------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| [147](https://atomgit.com/opengauss/oGRAC/issues/147) | oGRAC | [Bug]: 创建外键约束表数据库core |
| [156](https://atomgit.com/opengauss/oGRAC/issues/156) | oGRAC | [Bug]: cms命令拉起指定节点ograc失败后会自动重拉失败 |
| [159](https://atomgit.com/opengauss/oGRAC/issues/159) | oGRAC | [Bug]: 【测试类型：功能测试】【测试版本：7.0.0 LTS】部署两节点，0节点部署成功，1节点install安装报错 |
| [160](https://atomgit.com/opengauss/oGRAC/issues/160) | oGRAC | [Bug]: 【测试类型：功能测试】【测试版本：7.0.0 LTS】部署两节点，执行业务或者空载，节点0/1异常挂掉 |
| [194](https://atomgit.com/opengauss/oGRAC/issues/194) | oGRAC | [Bug]: oGRAC 状态类/恢复类语句触发 UDS 断连、core 或节点异常 |
| [204](https://atomgit.com/opengauss/oGRAC/issues/204) | oGRAC | [Bug]: use_bison_parser=false 下同样的 SELECT 1 可以正常返回；use_bison_parser=true 后才触发 UDS 断连和节点异常。 |
| [203](https://atomgit.com/opengauss/oGRAC/issues/203) | oGRAC | [Bug]: RESTORE DATABASE 异常失败后导致 UDS 断连/core 或 node0 db UNKNOWN |
| [205](https://atomgit.com/opengauss/oGRAC/issues/205) | oGRAC | [Bug]: B007 开启 bison parser 后执行 SELECT CTE 聚合 JOIN 场景触发 UDS 断连并导致实例不可连接 |
| [219](https://atomgit.com/opengauss/oGRAC/issues/219) | oGRAC | [Bug]: fast count 优化误判 JSON_VALUE 依赖左侧行导致 core |
| [220](https://atomgit.com/opengauss/oGRAC/issues/220) | oGRAC | [Bug]: fast count 优化误判 JSON_TABLE data_expr 外部依赖导致 core |
| [272](https://atomgit.com/opengauss/oGRAC/issues/272) | oGRAC | [Bug]: 【测试类型：SQL语法】use_bison_parser = true;DDL语法-匿名块中RETURN 带值，挂库 |
| [279](https://atomgit.com/opengauss/oGRAC/issues/279) | oGRAC | [Bug]: 【测试类型：SQL语法】use_bison_parser = true;DDL创建对象，异常场景对象名称超长时执行挂库 |
| [293](https://atomgit.com/opengauss/oGRAC/issues/293) | oGRAC | [资料]: ogsql 自动补全文档缺少能力边界/负向说明（请补充 ogsql_instructions） |
| [288](https://atomgit.com/opengauss/oGRAC/issues/288) | oGRAC | [Bug]: ogsql输入select * fro按Tab错误补全为frozen_status |
| [299](https://atomgit.com/opengauss/oGRAC/issues/299) | oGRAC | [Bug]: ogsql 非 SYS 用户 schema. 后无法补全表名 |
| [297](https://atomgit.com/opengauss/oGRAC/issues/297) | oGRAC | [Bug]: ogsql 多行 continuation 下 FROM/WHERE Tab 补全失效 |
| [314](https://atomgit.com/opengauss/oGRAC/issues/314) | oGRAC | [Bug]: master分支跑ci用例产生core |
| [312](https://atomgit.com/opengauss/oGRAC/issues/312) | oGRAC | [Bug]: use_bison_parser=true，深嵌套用例core |
| [311](https://atomgit.com/opengauss/oGRAC/issues/311) | oGRAC | [Bug]: use_bison_parser=true，PLSQL中的|| null会core |
| [310](https://atomgit.com/opengauss/oGRAC/issues/310) | oGRAC | [Bug]: use_bison_parser=true，alter语句识别128M错误 |
| [321](https://atomgit.com/opengauss/oGRAC/issues/321) | oGRAC | [Bug]: 【测试功能：工具功能】cm磁盘检测，节点0关闭自动保护，构造共享盘满故障，恢复后节点1发生core dump |
| [329](https://atomgit.com/opengauss/oGRAC/issues/329) | oGRAC | [Bug]: 数据库重启后执行alter index rebuild发生core |
| [331](https://atomgit.com/opengauss/oGRAC/issues/331) | oGRAC | [Bug]: 【测试功能：工具功能】【双节点】节点0关闭读写自动切换，节点0、1分别kill掉dss进程，发生core dump |

### 继承功能

  - 基础功能：[SQL基础能力](../about_ograc/product_description/sql_ability.md#sql基础能力概述)、[SQL优化器](../about_ograc/product_description/sql_ability.md#sql优化器能力概述)、[事务管理](../about_ograc/product_architecture/transaction_mechanism.md#事务机制)等。
  - [多写能力](../about_ograc/product_architecture/technical_feature_overview_of_database_multi_write_capability.md#ograc引擎多写技术特性概述)
  - 存储引擎特性
    - [检查点机制](../about_ograc/product_architecture/ckpt.md)，在内存和磁盘之间建立一个同步点，以确保数据的一致性和持久性，并加速数据库的恢复过程。
    - [在线 DDL](../about_ograc/product_architecture/online_ddl.md)，在不影响或极小影响数据库正常提供服务的情况下，完成表定义的修改。
    - [redo 日志](../about_ograc/product_architecture/redo.md)，保证事务的持久性，即一旦事务提交，它对数据所做的修改就永久不会丢失，即使随后发生系统崩溃。
  - 可维护能力
    - [WSR报告](../about_ograc/maintainability.md#wsr报告)，通过对数据库打快照，利用两个不同时间点的生成的快照中收集到的统计数据，生成这期间的性能分析报告。
    - [统计视图](../about_ograc/maintainability.md#统计视图)，提供了多个视图供用户查看系统信息，按功能维度可以分为 DBA 视图、用户视图和性能视图，结合这些视图信息可以观察当前数据库内部的运行状态。
  - 应用驱动
    - [JDBC 驱动](../developer_guide/jdbc/jdbc_package_driver_class_and_environment_class.md#jdbc包驱动类和环境类)，JDBC（Java Database Connectivity，Java 数据库连接）是一种用于执行 SQL 语句的 Java API，可以为多种关系数据库提供统一访问接口，应用程序可基于它操作数据。
    - [ODBC 驱动](../developer_guide/odbc/linux_configure_data_source.md#linux下配置数据源)，Open Database Connectivity（ODBC）是一个被广泛接受的应用程序编程接口（API），用于数据库访问。它基于开放组和 ISO/IEC 中用于数据库 API 的 Call-Level 接口（CLI）规范，并使用结构化查询语言（SQL）作为其数据库访问语言。

### CVE漏洞

本版本是2026年9月发布的LTS版本，版本涉及的CVE漏洞可通过[CVE列表](https://opengauss.org/zh/cve)查询。

## 版本使用注意事项

- 版本技术规格可以参照《关于oGRAC》中的[规格](../about_ograc/specification/specification.md#表相关规格)。
- 可参照《安装指南》中的[单节点安装](../installation_guide/single_node_guide/local_installation_on_a_single_node.md)或[双节点安装](../installation_guide/two_nodes_guide/ograc_two_node_installation.md) ，选择业务需要的部署架构，保证数据库集群的可靠性和可用性。

## 源代码

oGRAC包含多个代码仓，包括引用开源软件的补丁代码仓、JDBC驱动代码仓、ODBC驱动代码仓、数据库服务器代码仓和文档仓库：

- 开源软件代码仓：[https://atomgit.com/opengauss/openGauss-third\_party](https://atomgit.com/opengauss/openGauss-third_party)
- JDBC驱动代码仓：[https://atomgit.com/opengauss/openGauss-connector-jdbc](https://atomgit.com/opengauss/openGauss-connector-jdbc)
- ODBC驱动代码仓：[https://atomgit.com/opengauss/openGauss-connector-odbc](https://atomgit.com/opengauss/openGauss-connector-odbc)
- 数据库服务器代码仓：[https://atomgit.com/opengauss/oGRAC](https://atomgit.com/opengauss/oGRAC)
- 数据库文档仓库：[https://atomgit.com/opengauss/oGRAC-doc](https://atomgit.com/opengauss/oGRAC-docs)

## 用户须知

oGRAC是一款支持多读多写的开源数据库。采用协议"木兰宽松许可证"（Mulan PSL V2），用户可以自由复制、使用、修改、分发，不论修改与否。

oGRAC 的版本号遵循 X.Y.0-RCx 的格式，旨在区分不同类型的版本更新。具体而言：

 + LTS 版本（X.0.0）：每两年发布一次，标志着长期稳定支持的版本，适合大规模部署与生产环境使用。社区承诺为每个 LTS 版本提供为期三年的维护支持，确保其持续稳定与安全。
 + 创新版本（X.Y.0-RCx）：每半年推出，旨在快速迭代，提供最新的功能与技术预览。这些版本主要用于用户测试与创新合作，社区将提供为期半年的维护支持，鼓励探索与反馈。
 + 补丁版本（X.Y.0）：当遇到重大问题时，会适时发布，旨在迅速修复关键性错误，保障系统稳定运行。

## 致谢

衷心感谢参与和协助 oGRAC 7.0.0-LTS 版本发布的所有开发者和伙伴。正是大家的辛勤付出使得版本顺利发布，也为 oGRAC 更好地发展提供了可能。
