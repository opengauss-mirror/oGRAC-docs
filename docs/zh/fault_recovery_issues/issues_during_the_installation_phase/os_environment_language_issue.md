# 操作系统语言为中文导致安装时无法获取 NUMA 信息卡住

## 一、问题现象
安装单节点 oGRAC 数据库，执行安装命令时卡住：

```
sh local_install.sh install -u ograc_user
```

命令回显一直在刷屏 Instance startup in progress, please wait：

```
用户id=1000(ogracadm) 组id=1000(ogracadm) 组=1000(ogracadm)
Checking runner.
Checking parameters.
End check parameters.
Checking old install.
End check old install.
Checking kernel parameters.
Checking directory.
Checking integrity of run file...
Decompressing run file.
Setting user env.
Checking data dir and config file
Initialize db instance.
Creating database with dbcompatibility 'A'.
RUN_MODE set to ogracd_in_cluster and made persistent.
Starting cms...
cms has started
Starting ogracd...
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
Instance startup in progress, please wait.
```

## 二、定位方法
查看数据库日志，发现有一行报错：

```
ERROR>g_cpu_group_num init error, g_cpu_group_num is 0
INFO>begin to shutdown, mode abort
INFO>begin to pause all listener
```

可以看出安装命令回显第一行是中文，可能为中文语言环境。查看当前语言设置：

```
localectl status
```

会显示类似：

```
System Locale: LANG=zh_CN.UTF-8
```

说明为中文环境。

## 三、问题根因

操作系统语言为中文，安装脚本用 lscpu 命令获取 NUMA 相关信息时，识别不了中文信息，因此获取信息到的 NUMA node 数量为 0，无法进行下一步。

## 四、解决方法

当前暂不支持中文环境安装，可临时将系统环境切换为英文，然后执行命令安装：

```
export LANG=en_US.UTF-8
```
