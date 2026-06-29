# 因两节点机器时间不同步导致安装双节点 oGRAC 失败的问题

## 一、问题现象

在安装双节点 oGRAC 过程中，依次完成节点 0 和节点 1 的预安装与安装后，启动节点 0 成功，但启动节点 1 时长时间卡住。随后登录节点 0 的安装用户，执行集群资源状态查询命令 `cms stat -res`，发现数据库与 DSS 资源均处于 offline 状态。

## 二、定位方法

使用`cd`命令进入节点 0 的`config_params_lun.json`配置文件中`module_config->ograc_home`配置项的路径目录下，然后再进入`log/ograc/run`目录下，查看`ogracd.rlog`日志，有以下日志：
```shell
ERROR>cms cli uds request failed [cms_interface.c:1014]
ERROR>socket send failed,uds conn is closed, sock -1, session id xxx, msg type xxx, msg seq xxx, timout 1000 [cms_uds_client.c:214]
ERR0R>send hb msg failed [cms_interface.c:238]
ERROR>cms cli send hb failed [cms_interface.c:709]
ERROR> [CMS_CLI] ABORT INFO: cms cli conn retry failed [cms_interface.c:696]
```
从上述日志得知，节点 0 已经无法连接到 CMS，再往时间最接近的日志中查看，发现以下日志：
```shell
ERROR>send hb msg failed [cms_interface.c:238]
ERROR> cms cli send hb failed [cms_interface.c:709]
......
ERROR>cms cli retry conn succ, i 2 [cms_interface.c:689]
ERROR>[RC] ABORT INFO: self abort, notified by CMS kick-out from cluster, version is 12. [rc_refrom.c:545]
```
从上述日志得知，节点 0 被CMS踢出集群，导致节点 0 数据库进程不存在，变为了 offline 状态。接着，使用使用`cd`命令进入节点 0的`log/cms/run`目录下，查看`cms_srv.rlog`日志，有以下日志：
```shell
INFO>cms node[0] vote time is 2026-06-24 20:55:53.275, curr round start time is = 2026-06-24 20:55:43.104, curr round end time is = 2026-06-24 20:55:47.104 [cms_vote.c:773]
......
INFO>cms node[1] vote time is 2026-06-24 20:55:43.104, curr round start time is = 2026-06-24 20:55:43.104, curr round end time is = 2026-06-24 20:55:47.104 [cms_vote.c:773]
```
从上述日志得知，节点 0 的投票时间为2026-06-24 20:55:53.275，而节点 1 的投票时间为2026-06-24 20:55:43.104，并且发现当前投票轮的起止时间为2026-06-24 20:55:43.104至2026-06-24 20:55:47.104，发现节点 0 的投票时间未在投票轮时间内，且与节点 1 的投票时间相差约 10 秒，因此怀疑是 0 节点机器时间太快，导致投票时间未在投票轮内，被判定为无效投票，进而导致节点 0 被CMS踢出集群，导致节点 0 数据库进程不存在，变为了 offline 状态。

## 三、问题根因

两节点系统时间不同步（相差约 10 秒），导致 CMS 投票算法误判，最终节点 0 被踢出集群。

## 四、解决方法

1. 保证两节点时间同步。（若使用虚拟机，需先关闭与宿主机的时间同步策略，防止出现时间跳变问题。）

    #### 情况一：节点可访问外网

    两节点分别执行：

    ```shell
    ntpdate -u [外网ntp服务器网址]
    ```

    #### 情况二：无外网环境

    * 节点 0 作为时间服务器
    * 节点 1 向节点 0 同步时间

    **节点 0：**

    ```shell
    sed -i "1i allow all" /etc/chrony.conf
    systemctl restart chronyd
    sed -i 's/^#local stratum 10/local stratum 10/' /etc/chrony.conf
    ss -unlp | grep chronyd
    ```

    **节点 1：**

    ```shell
    sed -i "1i server [节点1 IP地址] iburst" /etc/chrony.conf
    systemctl enable --now chronyd
    systemctl restart chronyd  #若后续由于其他因素导致时间偏差过大，可通过该命令快速触发强制同步
    chronyc tracking
    ```

2. 启动 oGRAC 前，先确认两节点时间差在 4 秒以内。

   ```bash
   # 节点 0
   date

   # 节点 1
   date
   ```

3. 修复时间同步后，重新执行完整的清理、安装与启动流程即可。
