# `DSS` `LUN` 注册冲突导致安装失败

## 现象描述

在两节点或多节点部署场景中，`DSS` 共享盘异常会导致安装失败，常见表现如下：

* 安装过程中的 `install` 或 `start` 阶段失败
* `DSS` 组件无法启动
* 在以下日志中出现异常信息：

```text
/opt/ograc/log/dss/run/instance.log
```

`DSS` 注册阶段失败时，`/opt/ograc/log/dss/run/instance.log` 中典型报错如下：

```text
ERROR [pid: 2127231] [MainThread] [tid:281460975927024] [dssctl.py:596] Reghl node cmd[source ~/.bashrc && /opt/ograc/dss/bin/dsscmd reghl -D /opt/ograc/dss] failed, details: Begin to register,
Failed to get vg non entry info when reghl, errcode is -1.
 detail reason[2031]: The volume group has not been initialized.
Failed to register.
```

## 常见原因

* 共享 `LUN` 已被其他集群或历史环境注册
* 先前安装未正常卸载，残留注册信息
* 其他业务对共享盘进行了 `Persistent Reservation`

## `DSS` `LUN` 注册冲突的排查与清理

### 查看 `LUN` 注册信息

```shell
sg_persist --in --read-keys /dev/xxx
```

若返回如下内容，说明该 `LUN` 已存在注册信息：

```text
PR generation=0xb6, 2 registered reservation keys follow:
0x1
0x2
```

### 清理注册信息

对每一个已注册的 key 执行清理操作：

```shell
sg_persist --out --clear --param-rk=<key> /dev/xxx
```

其中 `<key>` 为上一步查询到的 reservation key。

### 确认清理结果

再次执行：

```shell
sg_persist --in --read-keys /dev/xxx
```

若输出如下内容：

```text
there are NO registered reservation keys
```

说明共享盘注册信息已成功清理，可重新执行安装或启动流程。
