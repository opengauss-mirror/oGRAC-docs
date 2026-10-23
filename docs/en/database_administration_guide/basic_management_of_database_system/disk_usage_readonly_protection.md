# CMS Disk Usage Monitoring and Automatic Read-Only Protection

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-10-13T01:59:09.498Z pushedAt=2026-10-15T13:05:31.887Z -->

## 1. Feature Overview

The CMS disk usage monitoring and automatic read-only protection feature is used to monitor the usage of critical disk space in an oGRAC resource pooling cluster. When disk usage reaches the configured threshold, CMS can automatically switch the database to `readonly`, preventing the database from continuing to write and causing disk exhaustion, failure to flush logs to disk, data file extension failures, or cluster anomalies.

In resource pooling deployment scenarios, the database typically relies on shared storage and DSS to manage critical storage resources such as data disks, redo disks, and archive disks. Disk space exhaustion may affect the database write capability and cluster stability. Therefore, CMS periodically collects the space usage of the following objects and determines, based on the configuration, whether to trigger `readonly` or `readwrite` protection actions:

- The file system where the local database data directory is located.
- DSS VG.

### Main Capabilities

1. Periodically collect the disk usage rate of the file system where the local database data directory resides.
2. Periodically collect the disk usage rate of the DSS VG.
3. View the current disk usage rate, threshold, automatic protection switch, cooldown time, and read-only protection state through CMS commands.
4. When the disk usage rate reaches the configured threshold, automatically switch the database to `readonly`.
5. When the disk usage rate recovers to below the threshold and the cooldown time requirement is met, automatically restore the database to `readwrite`.
6. Dynamically modify the threshold, inspection period, cooldown time, and automatic protection switch through commands.
7. After modification, the parameters are written to the `cms.ini` configuration file.

## 2. Working Mechanism

After CMS starts, it creates a disk usage rate inspection thread. This thread performs disk inspections at the interval configured by `_DISK_USAGE_CHECK_INTERVAL`.

Each round of inspections mainly performs the following process:

1. Read the disk protection-related configuration in `cms.ini`.
2. Collect the capacity, used space, remaining space, and usage rate of the file system where the local database directory is located.
3. Attempt to discover the DSS environment, and collect the capacity, used space, remaining space, and usage rate of the DSS VG through `dsscmd`.
4. Generate a disk usage rate snapshot for querying with `cms diskreadonly -show`.
5. Determine whether there is a disk object whose collection succeeds and whose usage rate reaches the threshold.
6. If the automatic protection switch is enabled and the `readonly` trigger condition is met, send a `readmode switch` message to the database resource on the local node to switch the database to `readonly`.
7. If the database has been switched to `readonly` by this feature, and the disk usage rate drops below the threshold and meets the cooldown time requirement, the system attempts to automatically restore the database to `readwrite`.

### Automatic Readonly Determination Conditions

```text
Disk usage rate >= _DISK_USAGE_THRESHOLD
```

Only successfully collected disk objects participate in the trigger judgment. Objects whose collection fails are displayed as `ERROR` in the query result, but they do not trigger `readonly` as successful alarm objects.

## 3. Enabling and Disabling the Feature

### 3.1 Viewing the Current Feature Status

```bash
cms diskreadonly -show
```

Example output:

```text
DISK_USAGE_CHECK_INTERVAL = 30
DISK_USAGE_THRESHOLD = 85
DISK_USAGE_PROTECT_ENABLE = TRUE
DISK_USAGE_READONLY_COOLDOWN = 120
DISK_USAGE_READONLY_STATE = NORMAL
TYPE   NAME   SOURCE                           TOTAL_GB   USED_GB   FREE_GB   USE%     THRESHOLD   STATUS   LAST_CHECK            INFO
LOCAL  local  /home/ogracdba/data/data         100.000    40.000    60.000    40.000   85          OK       2026-08-05 12:00:00  OK
DSS    data   /opt/ograc/dss/bin/dsscmd        2048.000   1200.000  848.000   58.594   85          OK       2026-08-05 12:00:00  discovered from environment
```

Note:

- `cms diskreadonly -show` queries the latest disk usage rate snapshot in the CMS memory.
- This command does not directly read the `cms.ini` file.
- After parameters are modified, they are written to `cms.ini`, but the result of `-show` may need to wait for the next inspection cycle before it is refreshed.
- The default inspection cycle is 30 seconds. Therefore, after parameters are modified, there may be a display delay of at most one inspection cycle.

### 3.2 Enable Automatic Read-Only Protection

```bash
cms diskreadonly -auto_change True
```

After successful execution, the configuration in `cms.ini` is updated to:

```ini
_DISK_USAGE_PROTECT_ENABLE = TRUE
```

After it is enabled, when disk usage reaches the threshold, CMS automatically triggers the database to switch to `readonly`.

### 3.3 Disabling Automatic Read-only Protection

```bash
cms diskreadonly -auto_change False
```

After successful execution, the configuration in `cms.ini` is updated to:

```ini
_DISK_USAGE_PROTECT_ENABLE = FALSE
```

After disabling this function, CMS still collects and displays the disk usage rate, but does not automatically trigger `readonly`/`readwrite` switching.

> Note: Disabling automatic read-only protection does not automatically restore the database to `readwrite`. If the database has already been switched to `readonly`, confirm that the disk space and database state meet the recovery conditions, and then manually execute the recovery command.

## 4. Common Operations

### 4.1 Viewing Disk Usage Rate and Protection Status

```bash
cms diskreadonly -show
```

Used to view the current disk usage rate, protection configuration, and read-only protection state.

### 4.2 Setting the Disk Usage Rate Threshold

```bash
cms diskreadonly -set threshold 90
```

Indicates that when the usage rate of any successfully collected disk object reaches 90%, the `readonly` protection inspection is triggered.

After modification, the configuration in `cms.ini` is:

```ini
_DISK_USAGE_THRESHOLD = 90
```

### 4.3 Setting the Automatic Switch Cooldown Time

```bash
cms diskreadonly -set cooldown 300
```

Indicates that the interval between two `readonly`/`readwrite` actions is at least 300 seconds.

After modification, the configuration in `cms.ini` is as follows:

```ini
_DISK_USAGE_READONLY_COOLDOWN = 300
```

### 4.4 Setting the Disk Usage Rate Polling Interval

```bash
cms diskreadonly -set interval 60
```

Indicates that CMS collects the disk usage rate every 60 seconds.

After modification, the configuration in `cms.ini` is as follows:

```ini
_DISK_USAGE_CHECK_INTERVAL = 60
```

### 4.5 Manually Restoring the Database to readwrite

```bash
cms diskreadonly -recover-now
```

This command is used to manually trigger the database to be restored to `readwrite`.

Applicable scenarios:

1. The database has been automatically switched to `readonly` due to disk protection.
2. O&M personnel have completed disk cleanup.
3. It has been confirmed that the disk space has recovered to a safe range.
4. It has been confirmed that the database can resume writing.

Note:

- If the database is already in `readwrite`, the command returns information indicating that the database is already in `readwrite` or that the recovery is successful.
- If the database resource is not registered, the communication between CMS and the database fails, or the database state does not meet the recovery conditions, the recovery fails.
- Manual recovery of `readwrite` will refresh the latest action time, and subsequent automatic switching is still subject to the `cooldown` limit.

## 5. Command Summary

| Command | Function Description | Whether to Write to `cms.ini` | Remarks |
| --- | --- | --- | --- |
| `cms diskreadonly -show` | View the disk usage rate and read-only protection configuration snapshot | No | Queries the CMS memory snapshot and does not directly read the configuration file |
| `cms diskreadonly -auto_change True` | Enable automatic `readonly`/`readwrite` protection | Yes | Writes `_DISK_USAGE_PROTECT_ENABLE = TRUE` |
| `cms diskreadonly -auto_change False` | Disable automatic `readonly`/`readwrite` protection | Yes | Writes `_DISK_USAGE_PROTECT_ENABLE = FALSE` |
| `cms diskreadonly -set threshold <percent>` | Set the disk usage rate protection threshold | Yes | Writes `_DISK_USAGE_THRESHOLD` |
| `cms diskreadonly -set cooldown <seconds>` | Set the `readonly`/`readwrite` action cooldown time | Yes | Writes `_DISK_USAGE_READONLY_COOLDOWN` |
| `cms diskreadonly -set interval <seconds>` | Set the disk usage rate inspection interval | Yes | Writes `_DISK_USAGE_CHECK_INTERVAL` |
| `cms diskreadonly -recover-now` | Manually restore the database to `readwrite` | No | Used to manually restore write access after disk cleanup |

## 6. Querying Result Field Description

`cms diskreadonly -show` displays the disk protection configuration and disk usage details.

### 6.1 Configuration Fields

| Field | Meaning | Example |
| --- | --- | --- |
| `DISK_USAGE_CHECK_INTERVAL` | Disk usage rate inspection interval, in seconds | `30` |
| `DISK_USAGE_THRESHOLD` | Disk usage rate protection threshold, in percentage | `85` |
| `DISK_USAGE_PROTECT_ENABLE` | Whether automatic `readonly`/`readwrite` protection is enabled | `TRUE` |
| `DISK_USAGE_READONLY_COOLDOWN` | Cooldown time for `readonly`/`readwrite` actions, in seconds | `120` |
| `DISK_USAGE_READONLY_STATE` | Current automatic read-only protection state | `NORMAL` |

### 6.2 Disk Detail Fields

| Field | Meaning |
| --- | --- |
| `TYPE` | Type of the disk object. `LOCAL` indicates a local database directory, and `DSS` indicates a DSS VG. |
| `NAME` | Name of the disk object. |
| `SOURCE` | Source of the disk object. For a local disk, the data directory path is displayed. For DSS, the `dsscmd` path is displayed. |
| `TOTAL_GB` | Total capacity, in GB. |
| `USED_GB` | Used capacity, in GB. |
| `FREE_GB` | Free capacity, in GB. |
| `USE%` | Current disk usage rate. |
| `THRESHOLD` | Protection threshold used by the current object. |
| `STATUS` | Status of the current object. `OK` indicates normal, `WARN` indicates that the threshold is reached, and `ERROR` indicates a collection failure. |
| `LAST_CHECK` | Time of the last collection. |
| `INFO` | Collection description or error information. |

## 7. Parameter Details

### 7.1 `_DISK_USAGE_CHECK_INTERVAL`

| Item | Description |
| --- | --- |
| Parameter meaning | CMS disk usage rate inspection interval |
| Parameter type | Integer |
| Unit | Second |
| Value range | `5 ~ 3600` |
| Default value | `30` |
| Whether command modification is supported | Supported |
| Modification command | `cms diskreadonly -set interval <seconds>` |
| Whether to write to `cms.ini` | Yes |

Example:

```bash
cms diskreadonly -set interval 60
```

Description:

- Controls how often CMS collects the disk usage rate.
- The inspection objects include the file system where the local database directory is located and the DSS VG.
- A smaller value enables more timely awareness of disk water level changes, but increases the collection frequency.
- A larger value has less impact on the system, but disk water level changes are detected more slowly.
- After modification, the value is written to `cms.ini`.
- `cms diskreadonly -show` queries the CMS memory snapshot. After modification, the new value may not be displayed until the next inspection round.

### 7.2 `_DISK_USAGE_THRESHOLD`

| Item | Description |
| --- | --- |
| Parameter meaning | Disk usage rate protection threshold |
| Parameter type | Integer |
| Unit | Percentage |
| Value range | `1 ~ 100` |
| Default value | `85` |
| Whether command modification is supported | Supported |
| Modification command | `cms diskreadonly -set threshold <percent>` |
| Whether to write to `cms.ini` | Yes |

Example:

```bash
cms diskreadonly -set threshold 90
```

Description:

- When any successfully collected disk object meets `USE% >= _DISK_USAGE_THRESHOLD`, CMS considers that the object reaches the protection condition.
- Both the local database directory and the DSS VG use this threshold for judgment.
- Only successfully collected disk objects participate in the `readonly` trigger judgment.
- Disk objects whose collection fails are displayed as `ERROR`, but they do not trigger `readonly` as successful alarm objects.
- A threshold that is too low may cause the database to enter `readonly` prematurely.
- A threshold that is too high may cause disk space risks to be discovered too late.
- It is recommended to set this value based on the service write speed, redo generation speed, archive growth speed, and O&M response time.

### 7.3 `_DISK_USAGE_PROTECT_ENABLE`

| Item | Description |
| --- | --- |
| Parameter meaning | Whether to enable automatic `readonly`/`readwrite` protection for disk usage rate |
| Parameter type | Boolean |
| Value range | `TRUE`, `FALSE` |
| Default value | `TRUE` |
| Whether command modification is supported | Supported |
| Modification command | `cms diskreadonly -auto_change True/False` |
| Whether to write to `cms.ini` | Yes |

Enable:

```bash
cms diskreadonly -auto_change True
```

Disable:

```bash
cms diskreadonly -auto_change False
```

Note:

- When set to `TRUE`, CMS automatically triggers `readonly` after the disk usage rate reaches the threshold.
- When set to `FALSE`, CMS still collects the disk usage rate but does not automatically trigger the `readonly`/`readwrite` switch.
- This parameter only controls the automatic protection action and does not affect the query capability of `cms diskreadonly -show`.
- If the database is already in `readonly`, disabling this parameter does not automatically restore `readwrite`.
- To restore `readwrite`, use `cms diskreadonly -recover-now` or restore it through other database management methods.

### 7.4 `_DISK_USAGE_READONLY_COOLDOWN`

| Item | Description |
| --- | --- |
| Parameter meaning | Cooldown time for `readonly`/`readwrite` actions |
| Parameter type | Integer |
| Unit | Second |
| Value range | `1 - 3600` |
| Default value | `120` |
| Whether command modification is supported | Supported |
| Modification command | `cms diskreadonly -set cooldown <seconds>` |
| Whether to write to `cms.ini` | Yes |

Example:

```bash
cms diskreadonly -set cooldown 300
```

Description:

- Used to prevent the database from frequently switching between `readonly` and `readwrite`.
- CMS records the time of the latest `readonly`/`readwrite` action.
- When the `cooldown` time has not elapsed, the action is not executed immediately even if the switching condition is met.
- Manual recovery of `readwrite` will refresh the latest action time.
- The cooldown time is calculated based on the time of the latest `readonly`/`readwrite` action, rather than only on the time when `readonly` was first triggered.
- A `cooldown` that is too short may cause the database read/write state to change frequently.
- An excessively long `cooldown` may cause the database to remain `readonly` for a long time after the disk recovers.

## 8. Read-Only Protection Status

`DISK_USAGE_READONLY_STATE` is used to display the current automatic read-only protection state.

| Status | Meaning |
| --- | --- |
| `NORMAL` | `readonly` is not currently triggered, the disk usage rate has not reached the protection condition, or the protection has been restored |
| `READONLY_PENDING` | The `readonly` condition has been reached, but the cooldown time has not elapsed, so `readonly` is not executed yet |
| `READONLY_TRIGGERED` | `readonly` has been triggered, and the database has been switched to or remains in `readonly` |
| `READONLY_FAILED` | The attempt to switch to `readonly` failed |
| `READWRITE_PENDING` | The disk has recovered, but the cooldown time has not elapsed, so `readwrite` is not automatically restored yet |
| `READWRITE_FAILED` | The attempt to restore `readwrite` failed |

## 9. Disk Object Status

The `STATUS` field is used to display the collection and alarm state of each disk object.

| Status | Meaning | Whether it participates in `readonly` triggering |
| --- | --- | --- |
| `OK` | Collection succeeded, and the disk usage rate has not reached the threshold | No |
| `WARN` | Collection succeeded, and the disk usage rate has reached or exceeded the threshold | Yes |
| `ERROR` | Collection failed | No |

Note:

- `WARN` indicates that the disk object has reached the protection threshold.
- `ERROR` indicates a collection failure. Check the local database process, the DSS environment, `DSS_HOME`, `dsscmd`, or the permission configuration.
- An `ERROR` object does not trigger `readonly` as a successful alarm object, but O&M personnel need to investigate the cause of the collection failure.

## 10. Automatic readonly Trigger Logic

Automatic `readonly` requires all of the following conditions to be met:

| Condition | Description |
| --- | --- |
| Automatic protection switch is enabled | `_DISK_USAGE_PROTECT_ENABLE = TRUE` |
| Inspection thread runs normally | The CMS disk usage rate inspection thread has been started |
| At least one object is collected successfully | At least one object of the local database directory or DSS VG is collected successfully |
| Disk usage reaches the threshold | Any successfully collected object meets `USE% >= _DISK_USAGE_THRESHOLD` |
| Not currently in the triggered state | The current state is not `READONLY_TRIGGERED` |
| Cooldown time requirement | The time since the last `readonly`/`readwrite` action exceeds `_DISK_USAGE_READONLY_COOLDOWN` |

After the preceding conditions are met, CMS sends a `readmode switch` message to the database resource on the local node to switch the database to `readonly`.

## 11. Automatic Readwrite Recovery Logic

Automatic recovery to `readwrite` requires that all of the following conditions are met at the same time:

| Condition | Description |
| --- | --- |
| `readonly` was previously triggered by this feature | The database `readonly` state was triggered by the disk protection feature |
| Automatic protection switch is enabled | `_DISK_USAGE_PROTECT_ENABLE = TRUE` |
| Disk usage rate has recovered | None of the successfully collected disk objects reaches the threshold |
| Triggering objects have recovered | All disk objects that originally triggered `readonly` have recovered to below the threshold |
| Cooldown time requirement | The time since the last `readonly`/`readwrite` action exceeds `_DISK_USAGE_READONLY_COOLDOWN` |
| Database state allows recovery | The current database state meets the `readwrite` recovery condition |

After the above conditions are met, CMS automatically attempts to recover the database to `readwrite`.

If you do not want to wait for automatic recovery, you can execute the following command after confirming that the disk space has been cleaned up and the database meets the recovery conditions:

```bash
cms diskreadonly -recover-now
```

## 12. Cooldown Time Description

The cooldown time is used to limit the execution frequency of `readonly`/`readwrite` actions, preventing the database read/write state from switching frequently.

### Example Scenario

| Time Point | Operation or Status | Whether to Refresh the Cooldown Time |
| --- | --- | --- |
| `T1` | Disk usage reaches the threshold, and CMS automatically switches the database to `readonly` | Yes |
| `T2` | The `cooldown` time is not exceeded, and the user manually recovers `readwrite` | Yes |
| `T3` | Disk usage reaching the threshold is detected again | Re-evaluate whether the `cooldown` after `T2` is exceeded |
| `T4` | The `readonly` condition is still met after the `cooldown` is exceeded | `readonly` can be triggered again |

Note:

- The cooldown time is calculated based on the time of the latest `readonly`/`readwrite` action.
- Manually recovering `readwrite` refreshes the latest action time.
- When the `cooldown` is not exceeded, the system enters the `pending` state even if disk usage reaches the threshold, and does not switch immediately.
- This mechanism prevents the database from repeatedly switching between read and write states when disk usage fluctuates around the threshold.

## 13. Configuration File Example

Example of related configuration in `cms.ini`:

```ini
_DISK_USAGE_CHECK_INTERVAL = 30
_DISK_USAGE_THRESHOLD = 85
_DISK_USAGE_PROTECT_ENABLE = TRUE
_DISK_USAGE_READONLY_COOLDOWN = 120
```

| Configuration Item | Example Value | Meaning |
| --- | ---: | --- |
| `_DISK_USAGE_CHECK_INTERVAL` | `30` | CMS inspections the disk usage rate every 30 seconds. |
| `_DISK_USAGE_THRESHOLD` | `85` | Protection judgment is triggered when the disk usage rate reaches 85%. |
| `_DISK_USAGE_PROTECT_ENABLE` | `TRUE` | Enable automatic `readonly`/`readwrite` protection. |
| `_DISK_USAGE_READONLY_COOLDOWN` | `120` | The interval between `readonly`/`readwrite` actions is at least 120 seconds. |



