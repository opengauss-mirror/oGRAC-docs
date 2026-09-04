# oGRAC Two-Node Multi-Write Test Demo

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:49:04.173Z pushedAt=2026-08-17T00:46:29.687Z -->

This document describes how to use the built-in test demo script to test the multi-write functionality of a two-node cluster after completing the installation and deployment of an `oGRAC` two-node cluster.

- Executing this script requires entering the oGRAC system administrator password set during installation, as well as the username to be created during the test execution (do not use a username that already exists in the database system).
- After executing `prepare`, a test user and a target test table are automatically created in the oGRAC database, and a background loop operation is started that performs `insert` operations on the target test table on both nodes simultaneously. The specific operation is as follows: 25 rows of data are inserted into both nodes simultaneously every 0.25 seconds, meaning 200 rows of data are inserted into the target test table per second.
- Once `prepare` finishes, executing `runtest` triggers a loop in which the script queries the total row count of the target test table on both nodes once per second. Each query looks back over the previous 4 seconds, so the returned row count typically falls between 600 and 800.
- After the test is complete, run `stop` to terminate the test, because a background write loop process was started during the `prepare` phase.
- You do not need to manually enter node IP addresses. The script automatically obtains the local and peer node IP addresses.
- The script uses the default port 1611 for database connection operations.

---

## Prerequisites

- The oGRAC two-node cluster has been installed by referring to [oGRAC Two-Node Deployment](./ograc_two_node_installation.md).

## Test Execution

### 1. Logging In to Any Node in the Cluster and Running the Following Commands

```shell
su -s /bin/bash ograc
cd /opt/ograc/ograc/server/admin/scripts/
# SYS_PASSWORD is the system administrator password entered during oGRAC installation. TEST_USER_NAME is the new username created when running this multi-write test demo. Do not use a user that already exists in the database system.
sh multi_master_test.sh <SYS_PASSWORD> <TEST_USER_NAME> --prepare
sh multi_master_test.sh <SYS_PASSWORD> <TEST_USER_NAME> --runtest
```

After executing the above test, the following output will be displayed on the screen:

```shell
Checking cluster status...
✓ Cluster status: OK
Local node IP: x.x.x.1
Remote node IP: x.x.x.2
Executing read test mode...
Initiating read test...
Read test log: /tmp/db_test_logs/read_test.log
Press Ctrl+C to terminate the read test
Start time: Wed Feb  4 08:32:46 PM CST 2026
[2026-02-04 20:32:46] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:42 to 2026-02-04 20:32:46
[2026-02-04 20:32:47] Local node: 625, Remote node: 625, Time range: 2026-02-04 20:32:43 to 2026-02-04 20:32:47
[2026-02-04 20:32:48] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:44 to 2026-02-04 20:32:48
[2026-02-04 20:32:49] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:45 to 2026-02-04 20:32:49
[2026-02-04 20:32:50] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:46 to 2026-02-04 20:32:50
[2026-02-04 20:32:52] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:47 to 2026-02-04 20:32:51
[2026-02-04 20:32:53] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:49 to 2026-02-04 20:32:53
[2026-02-04 20:32:54] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:50 to 2026-02-04 20:32:54
[2026-02-04 20:32:55] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:51 to 2026-02-04 20:32:55
[2026-02-04 20:32:56] Local node: 625, Remote node: 625, Time range: 2026-02-04 20:32:52 to 2026-02-04 20:32:56
[2026-02-04 20:32:57] Local node: 650, Remote node: 650, Time range: 2026-02-04 20:32:53 to 2026-02-04 20:32:57
......
```

This indicates that when both nodes in the cluster perform writes simultaneously, the data queried from both nodes is consistent in real time.

### 2. Stopping the Test

```shell
# If you are not logged in as the ograc user, switch to that user. If you are already logged in as the ograc user, no switch is needed.
su -s /bin/bash ograc
cd /opt/ograc/ograc/server/admin/scripts/
sh multi_master_test.sh --stop
```
