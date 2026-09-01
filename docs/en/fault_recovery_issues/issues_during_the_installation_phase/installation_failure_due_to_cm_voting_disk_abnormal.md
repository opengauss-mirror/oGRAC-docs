# Installation Failure Due to Abnormal CM Voting Disk (`gcc-disk`)

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:23.746Z pushedAt=2026-08-17T00:46:29.654Z -->

## Symptom

In two-node or multi-node deployment scenarios, shared storage is the most common source of installation failures, and related issues are typically concentrated in two categories: the **`CM` voting disk** and the **`DSS` shared disk**.

This issue is primarily manifested as:

* Failure during the **`CM` phase** of the installation process.
* Abnormal exit of both nodes during `CM` initialization or startup.
* Heartbeat write failures and `load disk` errors may appear in logs.

When the `CM` phase fails during installation, a typical error in `/opt/ograc/log/cms/cms_deploy.log` is as follows:

```text
Exception: failed to set cms node information.
command: sh /opt/ograc/action/cms/start_cms.sh -P install cms > /opt/ograc/log/cms/cms_deploy.log 2>&1
output:
Execute cms/install.sh cmsctl.py install failed
```

## Common Causes

* The symbolic link for `gcc-disk` is incorrect.
* The `gcc-disk` on both nodes **does not point to the same shared disk**.
* The voting disk cannot be written to normally during the erasure or initialization phase.

## Troubleshooting

1. Verify that the `gcc-disk` on both nodes points to the same `LUN`.
2. Use stable paths such as `/dev/disk/by-id` to recreate the symbolic link.
3. Verify that the voting disk is not occupied by other services.

> **Note**: The `CM` voting disk is used for cluster arbitration. Once it becomes abnormal, the cluster will fail to start normally.
