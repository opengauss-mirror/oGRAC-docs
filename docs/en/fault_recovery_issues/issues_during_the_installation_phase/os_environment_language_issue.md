# Installation Stuck When Retrieving NUMA Information Due to Chinese OS Locale

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:41.812Z pushedAt=2026-08-17T00:46:29.661Z -->

## 1. Symptom

When installing a single-node oGRAC database, the installation command hangs during execution:

```sh
sh local_install.sh install -u ograc_user
```

The command output continuously refreshes with "Instance startup in progress, please wait":

```text
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

## 2. Issue Locating

Check the database log. An error line is found:

```text
ERROR>g_cpu_group_num init error, g_cpu_group_num is 0
INFO>begin to shutdown, mode abort
INFO>begin to pause all listener
```

It can be seen that the first line of the installation command output is in Chinese, which may indicate a Chinese locale. Check the current language settings:

```sh
localectl status
```

An output similar to the following is displayed:

```sh
System Locale: LANG=zh_CN.UTF-8
```

This indicates a Chinese locale.

## 3. Root Cause

The operating system language was set to Chinese. When the installation script used the `lscpu` command to obtain NUMA-related information, it was unable to parse the Chinese output, resulting in the number of NUMA nodes being retrieved as 0, which prevented the installation from proceeding to the next step.

## 4. Solution

Installation in a Chinese locale is currently not supported. As a temporary workaround, switch the system language to English and then execute the installation command:

```sh
export LANG=en_US.UTF-8
```
