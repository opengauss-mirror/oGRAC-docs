# oGRAC Uninstallation Failure Caused by Residual Processes

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:54.682Z pushedAt=2026-08-17T00:46:29.667Z -->

**Symptom**

In a shared environment, after the first oGRAC cluster was uninstalled, an attempt to uninstall the second oGRAC environment by executing the `sh appctl.sh uninstall override` command failed, with the following error message on the console:

```text
RuntimeError: ograc process still running, stop before uninstall
[ERROR] Uninstall ograc failed
```

![ResidualProcesses](./ResidualProcesses.jpg)

**Cause Analysis**

1. **Residual processes**: During the uninstallation of the first oGRAC environment, some `ograc`, `dss`, or `cms` related processes may remain running in the background due to abnormal termination or incomplete script cleanup, occupying system resources (such as ports, shared memory, file locks, etc.).
2. **Environment conflicts**: In a shared environment, if the installation directory, user, port, or shared memory key (`_SHM_KEY`) of the second oGRAC environment overlaps or conflicts with that of the first environment, the uninstallation script will proactively abort the operation upon detecting active processes to prevent data corruption.
3. **Shared storage residuals**: If the first environment uses a shared LUN and the DSS persistent registration information (Persistent Reservation) is not fully cleaned up, it may cause the second environment to misjudge the process status during uninstallation.

**Solution**

1. **Manually stop residual processes**:

    ```sh
    # Switch to the root user.
    # Check and terminate related processes.
    ps -ef | grep -E 'ograc|dss|cms' | grep -v grep 
    kill -9 <PID>  
    # Terminate the found processes one by one.
    ```

2. **Clean up shared memory and semaphores**:

    ```sh
    # View and clean up shared memory segments (verify the _SHM_KEY value).
    ipcs -m | grep ograc 
    ipcrm -m <shmid> 
    # Clean up semaphores.
    ipcs -s | grep ograc 
    ipcrm -s <semid>
    ```

3. **Forcefully clean up the environment (proceed with caution)**:

    ```sh
    # Delete the installation directory and user (ensure data is backed up).
    rm -rf /opt/ograc /data/ograc_install 
    userdel -r ograc 
    groupdel ogdba 
    # Clean up shared disk registration information.
    sg_persist --out --clear --param-rk=<key> /dev/dss-disk*
    ```

4. **Retry uninstallation**: After completing the cleanup above, re-execute the uninstallation command.

**Preventive Measures**

- After uninstalling the first environment, you must use commands such as `ps` and `ipcs` to confirm that no related residual processes remain.
- When deploying multiple oGRAC instances in a shared environment, you must assign independent installation directories, users, ports, and `_SHM_KEY` values (by modifying `config_params_lun.json`) to each instance.
- Before uninstalling, execute `sh appctl.sh stop` first to ensure that the cluster is completely stopped.
