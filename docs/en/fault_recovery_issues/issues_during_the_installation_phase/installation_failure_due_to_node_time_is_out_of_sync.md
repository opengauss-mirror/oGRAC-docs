# Two‑Node oGRAC Installation Failure Due to Time Synchronization Issues Between Nodes

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:39.785Z pushedAt=2026-08-17T00:46:29.659Z -->

## 1. Symptom

During the installation of a two-node oGRAC cluster, after completing the pre-installation and installation of node 0 and node 1 in sequence, node 0 started successfully, but node 1 hung for an extended period during startup. Subsequently, logging in as the installation user on node 0 and executing the cluster resource status query command `cms stat -res` revealed that both the database and DSS resources were in the offline state.

## 2. Issue Locating

Use the `cd` command to navigate to the directory specified by the `module_config->ograc_home` configuration item in the `config_params_lun.json` configuration file on node 0, then enter the `log/ograc/run` directory and view the `ogracd.rlog` log file. The following log entries were found:

```shell
ERROR>cms cli uds request failed [cms_interface.c:1014]
ERROR>socket send failed,uds conn is closed, sock -1, session id xxx, msg type xxx, msg seq xxx, timout 1000 [cms_uds_client.c:214]
ERR0R>send hb msg failed [cms_interface.c:238]
ERROR>cms cli send hb failed [cms_interface.c:709]
ERROR> [CMS_CLI] ABORT INFO: cms cli conn retry failed [cms_interface.c:696]
```

From the above logs, it can be seen that node 0 was no longer able to connect to CMS. Upon examining the logs closest in time, the following entries were discovered:

```shell
ERROR>send hb msg failed [cms_interface.c:238]
ERROR> cms cli send hb failed [cms_interface.c:709]
......
ERROR>cms cli retry conn succ, i 2 [cms_interface.c:689]
ERROR>[RC] ABORT INFO: self abort, notified by CMS kick-out from cluster, version is 12. [rc_refrom.c:545]
```

From the above logs, it can be seen that node 0 was evicted from the cluster by CMS, causing the database process on node 0 to become nonexistent and transition to the offline state. Next, use the `cd` command to navigate to the `log/cms/run` directory on node 0 and view the `cms_srv.rlog` log file. The following log entries were found:

```shell
INFO>cms node[0] vote time is 2026-06-24 20:55:53.275, curr round start time is = 2026-06-24 20:55:43.104, curr round end time is = 2026-06-24 20:55:47.104 [cms_vote.c:773]
......
INFO>cms node[1] vote time is 2026-06-24 20:55:43.104, curr round start time is = 2026-06-24 20:55:43.104, curr round end time is = 2026-06-24 20:55:47.104 [cms_vote.c:773]
```

From the above logs, it can be seen that the voting time of node 0 was 2026-06-24 20:55:53.275, while the voting time of node 1 was 2026-06-24 20:55:43.104. It was also found that the start and end times of the current voting round were from 2026-06-24 20:55:43.104 to 2026-06-24 20:55:47.104. The voting time of node 0 fell outside the voting round window and differed from that of node 1 by approximately 10 seconds. Therefore, it was suspected that the system time on node 0 was too fast, causing its voting time to fall outside the voting round and be deemed an invalid vote. As a result, node 0 was evicted from the cluster by CMS, causing the database process on node 0 to become nonexistent and transition to the offline state.

## 3. Root Cause

The system time of the two nodes was out of sync (with a difference of approximately 10 seconds), which caused the CMS voting algorithm to misjudge, ultimately resulting in node 0 being evicted from the cluster.

## 4. Solution

1. Ensure time synchronization between the two nodes. (If virtual machines are used, disable the time synchronization policy with the host machine first to prevent time jump issues.)

    ### Scenario 1: Nodes Can Access the External Network

    Run the following command on both nodes:

    ```shell
    ntpdate -u [External NTP server URL]
    ```

    ### Scenario 2: Offline Environment

    * Node 0 serves as the time server.
    * Node 1 synchronizes time from node 0.

    **Node 0:**

    ```shell
    sed -i "1i allow all" /etc/chrony.conf
    systemctl restart chronyd
    sed -i 's/^#local stratum 10/local stratum 10/' /etc/chrony.conf
    ss -unlp | grep chronyd
    ```

    **Node 1:**

    ```shell
    sed -i "1i server [Node 1 IP address] iburst" /etc/chrony.conf
    systemctl enable --now chronyd
    systemctl restart chronyd  #If the time deviation becomes excessive later due to other factors, use this command to quickly trigger a forced synchronization.
    chronyc tracking
    ```

2. Before starting oGRAC, verify that the time difference between the two nodes is within 4 seconds.

   ```bash
   # Node 0
   date

   # Node 1
   date
   ```

3. After the time synchronization is restored, re-execute the complete cleanup, installation, and startup process.
