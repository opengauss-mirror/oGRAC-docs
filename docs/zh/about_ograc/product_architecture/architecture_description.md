# 产品架构 

## 运行架构

![](public_sys-resources/running.png)

- oGRAC是基于共享存储的集群架构。
- 由cms集群管理组件，DB数据库实例组件以及DSS开源集群文件系统组件组成。

## 逻辑架构

![](public_sys-resources/logic_arch2.png)

- 当前支持的驱动有 `JDBC` 驱动、C 语言的 oGRAC 驱动和 `ODBC` 驱动，后续还会支持 `Go` 和 `Python` 驱动。
- oGRAC 的实例管理模块包括通信管理和服务调度，线程池管理，`Session` 会话池管理以及全局内存管理 `SGA` 和私有内存管理 `PGA`。
- `SQL` 引擎模块：支持词法解析，语法解析，语义解析，逻辑优化，物理优化，执行器以及存储过程。
- 存储引擎模块：表空间管理，堆表（`Heap`）管理，索引管理，`Page Buffer` 缓存管理，`DC` 元数据管理，事务管理，`Redo` 管理，`Checkpoint`。
- `DSS` 模块是基于 `LUN` 的高性能的共享集群文件系统。
- `DMS` 是多写共享集群服务，提供 `DRC` 分布式资源管理、`DLS` 分布式锁服务、`DCS` 分布式缓存服务能力。
- `CMS` 是集群管理组件。
- `CBB` 模块是供 `DSS` 和 `DMS` 用的基础能力库，包括通信，数据结构，安全，文件等。
