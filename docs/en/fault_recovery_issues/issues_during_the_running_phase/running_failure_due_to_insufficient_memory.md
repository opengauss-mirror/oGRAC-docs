# oGRAC Runtime Failure Due to Insufficient System Memory

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:49.409Z pushedAt=2026-08-17T00:46:29.666Z -->

## 1. Symptom

During the operation of a two-node oGRAC cluster, when executing the cluster resource status query command `cms stat -res`, it was found that the database and DSS resources on node 0 were both in the offline state.

## 2. Issue Locating

Use the `cd` command to navigate to the path specified by the `module_config->ograc_home` configuration item in the `config_params_lun.json` configuration file on node 0. Then enter the `log/deploy` directory and view the `deploy_daemon_ograc-xxx.log` log file. The log contains the following entries:

```shell
[ERROR] [3811831] [ograc_daemon.sh 115] [ograc daemon] The top5 processes that occupy memory are: xxx1 223628 17.21GB xxx2 3773178 10.49GB xxx3 758 0.22GB xxx4 3771943 0.18GB xxx5 3770285 0.12GB .
[ERROR] [ograc daemon] The top5 processes that occupy memory are: xxx1 223628 17.21GB xxx2 3773178 10.49GB xxx3 758 0.22GB xxx4 3771943 0.18GB xxx5 3770285 0.12GB .
RES_SUCCESS 
[ERROR] [3811831] [ograc_daemon.sh 120] [ograc daemon] CMS abort due to memory pressure, usage=96.00%, available=7.00%.
[ERROR] [ograc daemon] CMS abort due to memory pressure, usage=96.00%, available=7.00%.
```

The above log clearly shows the top five processes with the highest memory usage, along with their process IDs and memory consumption. It also explicitly indicates that CMS exited due to excessive memory pressure.

## 3. Root Cause

The daemon process `ograc_daemon.sh` implements a memory pressure protection mechanism during runtime. When it detects that the memory usage exceeds 95% or the available memory drops below 5%, it prints the top five processes with the highest memory consumption, disables the CMS auto-start feature, stops the CMS, and records the corresponding log entries.

## 4. Solution

After clearing memory to meet the required level, restart services:

```shell
# Start CMS
cms server -start &
# Start db
cms res -start db
```
