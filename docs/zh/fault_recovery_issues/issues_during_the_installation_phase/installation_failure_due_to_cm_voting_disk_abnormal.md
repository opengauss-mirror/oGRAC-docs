# `CM` 投票盘（`gcc-disk`）异常导致安装失败

## 现象描述

在两节点或多节点部署场景中，共享存储是最容易引发安装失败的环节，相关问题通常集中在 **`CM` 投票盘** 和 **`DSS` 共享盘** 两类。

本问题主要表现为：

* 安装过程中在 **`CM` 阶段失败**
* 两节点在初始化或启动 `CM` 时异常退出
* 日志中可能出现心跳写入失败、`load disk` 相关报错

`CM` 阶段安装失败时，`/opt/ograc/log/cms/cms_deploy.log` 中典型报错如下：

```text
Exception: failed to set cms node information.
command: sh /opt/ograc/action/cms/start_cms.sh -P install cms > /opt/ograc/log/cms/cms_deploy.log 2>&1
output:
Execute cms/install.sh cmsctl.py install failed
```

## 常见原因

* `gcc-disk` 软链接错误
* 两节点的 `gcc-disk` **未指向同一块共享盘**
* 投票盘在抹除或初始化阶段无法正常写入

## 排查与解决建议

1. 确认两节点 `gcc-disk` 指向同一 `LUN`
2. 使用 `/dev/disk/by-id` 等稳定路径重新建立软链接
3. 确认投票盘未被其他业务占用

> **提示**：`CM` 投票盘用于集群仲裁，一旦异常，集群将无法正常启动。
