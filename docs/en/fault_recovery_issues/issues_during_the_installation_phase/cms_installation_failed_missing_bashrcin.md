# CMS Installation Failure Due to Missing `.bashrc` File in Residual User Home Directory

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:18.551Z pushedAt=2026-08-17T00:46:29.650Z -->

## Symptom

During the deployment of oGRAC on two nodes, when executing `sh appctl.sh install config_params_lun.json`, the installation process was interrupted at the **CMS component installation phase**, with the following error message:

```text
[EROR]cms install failed: CMS install failed:[Errno 2] No such file or directory:'/home/ograc/.bashrc'
```

The installation script terminated automatically, and a complete oGRAC binary environment was not generated.

![Missing bashrc](./Missingbashrc.png)

## Common Causes

This issue is typically caused by **incomplete cleanup of residual files from previous installations**. Specific scenarios include:

1. During the previous uninstallation of oGRAC, the `ograc` user and its home directory were not completely removed using `userdel -r ograc`; only runtime directories such as `/opt/ograc` were manually deleted.
2. The residual home directory of the `ograc` user (`/home/ograc`) was retained, but environment configuration files such as `.bashrc` within the directory were lost or accidentally deleted.
3. During the current reinstallation, the installation script detected that the `ograc` user already exists and did not reinitialize the home directory environment, causing the CMS component to be unable to locate the `/home/ograc/.bashrc` file when configuring user environment variables.

## Troubleshooting

1. **Confirm the residual user status.**

   Execute the following command on both nodes to check whether the `ograc` user exists:

   ```bash
   id ograc
   ls -la /home/ograc/.bashrc
   ```

   If the user exists but the `.bashrc` file does not, this is the cause of the issue.

2. **Thoroughly clean up residual users and environment.**

   Before installation, ensure that all historical residues are completely removed (requires `root` user privileges):

   ```bash
   # Stop and uninstall the existing environment (if still running).
   sh appctl.sh stop
   sh appctl.sh uninstall override
   
   # Completely delete the ograc user and its home directory.
   userdel -r ograc
   # Verify that no other related residual users remain.
   userdel -r ogdba 2>/dev/null
   groupdel ogdba 2>/dev/null
   
   # Clean up any residual installation directories.
   rm -rf /opt/ograc /data/ograc_install
   ```

3. **Re-execute the installation process.**

   After the cleanup is complete, re-execute the pre-installation and installation commands. The script will automatically create a new `ograc` user and generate a complete home directory environment.

   ```bash
   sh appctl.sh pre_install config_params_lun.json
   sh appctl.sh install config_params_lun.json
   ```
