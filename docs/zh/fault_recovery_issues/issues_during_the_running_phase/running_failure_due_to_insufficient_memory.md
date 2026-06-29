# 因内存不足导致双节点 oGRAC 运行过程中 cms 挂掉的问题

## 一、问题现象

在双节点 oGRAC 运行过程中，执行集群资源状态查询命令 `cms stat -res`，发现 0 节点数据库与 DSS 资源均处于 offline 状态。

## 二、定位方法

使用`cd`命令进入节点 0 的`config_params_lun.json`配置文件中`module_config->ograc_home`配置项的路径目录下，然后再进入`log/deploy`目录下，查看`deploy_daemon_ograc-xxx.log`日志，有以下日志：
```shell
[ERROR] [3811831] [ograc_daemon.sh 115] [ograc daemon] The top5 processes that occupy memory are: xxx1 223628 17.21GB xxx2 3773178 10.49GB xxx3 758 0.22GB xxx4 3771943 0.18GB xxx5 3770285 0.12GB .
[ERROR] [ograc daemon] The top5 processes that occupy memory are: xxx1 223628 17.21GB xxx2 3773178 10.49GB xxx3 758 0.22GB xxx4 3771943 0.18GB xxx5 3770285 0.12GB .
RES_SUCCESS 
[ERROR] [3811831] [ograc_daemon.sh 120] [ograc daemon] CMS abort due to memory pressure, usage=96.00%, available=7.00%.
[ERROR] [ograc daemon] CMS abort due to memory pressure, usage=96.00%, available=7.00%.
```
上述日志可以清晰的看到占用内存最高的前五个进程是什么、进程号和内存占用情况，并且也有清晰的日志说明 CMS 因内存压力过大而退出。

## 三、问题根因

守护进程`ograc_daemon.sh`在运行过程中的内存压力保护，当发现内存占用率超过 95% 或可用低于 5%，则会打印当前占用内存最高的前五个进程，然后禁用 CMS 自启动，接着停掉 CMS 并记录下这段日志。

## 四、解决方法

针对该问题的解决办法为：清理内存至符合要求的大小后，重启即可，重启步骤如下：
```shell
# 启动 CMS
cms server -start &
# 启动 db
cms res -start db
```
