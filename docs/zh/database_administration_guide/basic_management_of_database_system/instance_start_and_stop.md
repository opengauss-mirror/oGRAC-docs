# 实例启停

在进行集群维护、故障排查和修复时，可以根据需要对集群和实例进行启停。

cms的各个命令需要切换集群用户后才能执行，请执行以下命令切换用户：

```
# su -s /bin/bash - ograc
```

## 命令说明

使用方法：

```
cms res -start
cms res -start RESOURCE_NAME
cms res -start RESOURCE_NAME -node NODE_ID
cms res -start RESOURCE_NAME TIMEOUT
cms res -stop RESOURCE_NAME
cms res -stop RESOURCE_NAME -node NODE_ID
```

选项参数：

| 参数            | 说明                        |
| ------------- | ------------------------- |
| RESOURCE_NAME | 资源名称，当前仅有db和dss资源         |
| NODE_ID       | 节点id，可通过 `cms stat -node` 命令查询获取 |
| TIMEOUT       | 启动超时阈值（启动时间超过阈值会返回，否则会等待，默认值为600秒） |

## 命令参考

1. 启动集群或实例

    **注意事项**

    - 执行该命令前，检查待启动节点的CMS server是否存活（执行 `ps -ef|grep 'cms server'` 命令查看是否存在cms server进程来判断）。

    - 启动指定节点的资源，默认等待时间为600秒，600秒仍未成功启动，该命令超时报错退出。

    启动cms资源（如需后台挂载，则使用 cms server -start &）：

    ```
    cms server -start
    start resource succeed.
    NODE_ID     = 0
    CMS_HOME    = /opt/oGRAC/data
    GCC_HOME    = /opt/oGRAC/data/gcc_home/gcc_file
    CMS GCC_TYPE:FILEVERSION     = oGRAC Debug 25.12 0d24758
    cms startup...
    MES: LSNR 127.0.0.1:23456
    ```

    启动集群内所有db节点：

    ```
    cms res -start db
    start resource succeed.
    ```

    启动指定db节点：

    ```
    cms res -start db -node 0
    start resource succeed.
    ```

    启动集群内所有dss节点：

    ```
    cms res -start dss
    start resource succeed.
    ```

    启动集群内指定dss节点：

    ```
    cms res -start dss -node 0
    start resource succeed.
    ```

    指定等待时间启动集群内节点：

    ```
    cms res -start db 120000
    start resource succeed.
    ```

2. 停止集群或实例
    停止cms资源：

    ```
    cms server -stop
    stop cms server succeed.
    ```

    停止集群内所有db节点：

    ```
    cms res -stop db
    stop resource succeed.
    ```

    停止指定db节点：

    ```
    cms res -stop db -node 0
    stop resource succeed.
    ```

    停止集群内所有dss节点：

    ```
    cms res -stop dss
    stop resource succeed.
    ```

    停止指定dss节点：

    ```
    cms res -stop dss -node 0
    stop resource succeed.
    ```
> [!WARNING]注意
>
> * DSS 基本介绍
>
> DSS（Distributed Storage Service）是 oGRAC 的分布式共享存储服务，负责管理共享磁盘卷组（VG）。它的核心职责是提供节点间共享存储的访问能力，管理 DSS 磁盘组（如 vg1、vg2），并为数据文件、日志文件提供底层 I/O 通路。DSS 通过 dssserver 等进程运行，其关键资源为裸设备或块设备映射的 DSS 卷组，可通过 dsscmd lsvg 查看。DSS 本身不解析 SQL，也不参与事务处理，仅提供块级或文件级的存储服务，属于基础设施层。
> * DB 基本介绍
>
> DB 即 oGRAC 数据库实例（ogracd），负责 SQL 解析、事务处理、缓存管理等核心数据库功能。它运行业务 SQL、管理 Buffer Pool、执行 DML/DDL 操作，并在 CM 管控下维护集群状态。DB 的进程主要为 ogracd，其关键资源包括表空间、数据文件和 WAL 日志。需要注意的是，DB 虽管理这些逻辑资源，但所有数据的实际落盘操作均由底层 DSS 完成。
> * 两者的关系
>
> DB 与 DSS 是底座与消费方的关系。DB 的数据文件、归档文件等均位于 DSS 管理的卷组中，DSS 是 DB 的存储底座，DB 是 DSS 的消费方。在启动顺序上，DB 启动时必须能正常访问 DSS 卷组，否则 DB 无法启动，因此须遵循"先 DSS，后 DB"的严格顺序。在通信层面，DB 通过 DSS 客户端库与 dssserver 交互，执行具体的 I/O 请求。在故障影响方面，一旦 DSS 故障，DB 会立即因 I/O 通路断开而失败或挂起；而 DB 故障通常不会直接影响 DSS 进程，但在停止操作时会触发级联行为。但若 CMS Server 进程仍存活，dssserver 进程会被自动拉起，因此建议在启动 DB 前使用`ps ux | grep dssserver | grep -v grep`确认 dssserver 进程是否存活。在功能定位层面，DSS 属于基础设施层，仅提供块级或文件级存储服务，完全不解析 SQL；DB 属于应用服务层，负责 SQL 处理、事务管理和缓存控制。
