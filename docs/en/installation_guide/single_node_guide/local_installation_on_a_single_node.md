# oGRAC Single-Node Local Installation

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:48:50.836Z pushedAt=2026-08-17T00:46:29.682Z -->

## Overview

This document guides developers through **single-node compilation, installation, and debugging** of oGRAC in a **local environment**. This deployment mode is primarily intended for functional verification, source code debugging, and development integration scenarios, and is not suitable for production environments.

> [!NOTE]
>
> * Single-node mode supports only single-instance operation.
> * Distributed component capabilities such as `DMS` and `DSS` are not included.
> * Multi-write scenarios are not supported, and the current environment cannot be smoothly scaled into a multi-node cluster.
> * Recommended only for development, debugging, and issue locating.

---

## Environment Setup

Before starting the installation, ensure that the operating system and runtime environment meet the following prerequisites:

1. The installation directory is restricted to `/home/` and must have sufficient space for installation (at least 20580 MB).
2. The operating system language must be set to English. Failure to do so will affect database startup.
3. The dedicated memory available for the database must be at least 16 GB.

### System Initialization

To prevent system security policies or firewall rules from affecting database processes, port communication, and debugging tools, SELinux and the firewall service must be disabled in advance.

Run the following commands to complete the relevant configuration:

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

> [!NOTE]
>
> * `setenforce 0`: Temporarily disables the SELinux enforcing policy.
> * Modifying `/etc/selinux/config` ensures that SELinux remains disabled after a system reboot.
> * After the firewall is disabled, ensure that the current environment is a trusted internal network or local environment.

---

### Creating Directories and Users

To ensure system security and permission isolation, it is recommended to use a **dedicated system user** for oGRAC installation and operation.

1. Create directories for source code compilation and installation.
2. Create a dedicated system user.
3. Grant directory permissions to the corresponding user.

Example:

```bash
mkdir -p [compile_path]
useradd [user_name]
passwd [user_password]
# Proper permission configuration is recommended; otherwise, a permission denied error may occur during the install phase.
chmod -R 777 [compile_path]
```

> [!NOTE]
>
> * `[compile_path]`: The unified working directory for source code download, compilation, and installation.
> * `[user_name]`: The system user dedicated to running oGRAC.

---

### Installing System Dependencies

The compilation and running of oGRAC depend on Python, network tools, and some basic libraries. Install the following packages in advance:

```bash
yum install -y wget python3 python3-devel iputils iproute \
unixODBC-devel unixODBC lz4 lz4-devel patch xz flex --skip-broken
```

> [!NOTE]
>
> * `python3 / python3-devel`: Used for executing installation scripts and management tools.
> * `iputils / iproute`: Used for network detection and IP configuration.
> * `lz4`: Used for data compression and decompression, version >= 1.8.3 required.
> * `--skip-broken`: Skips broken packages when dependency conflicts exist, preventing installation interruption.

---

## Source Code Acquisition and Compilation

### Obtaining Source Code

Switch to the compilation directory created earlier and pull the official oGRAC source code repository:

```bash
cd [compile_path]
git clone https://gitcode.com/opengauss/oGRAC.git
```

After the download is complete, the directory structure will contain core subdirectories such as `build` and `src`.

Then proceed with the prepare phase to install environment dependencies:

```bash
cd oGRAC/build
sh local_install.sh prepare
```

---

### Modifying Build Parameters

In certain debugging or development scenarios (especially during debug build), the **virtual memory protection mechanism** must be disabled; otherwise, it may affect the normal use of debugging tools.

Go to the build configuration directory and modify the parameters:

```bash
cd oGRAC/build
sed -i 's/DUSE_PROTECT_VM=ON/DUSE_PROTECT_VM=OFF/g' Makefile.sh
```

### Compilation

Compile in the `oGRAC/build` directory for subsequent binary installation:

```bash
sh local_install.sh compile -b [release | debug]
```

> [!NOTE]
> 
> * `[release | debug]`: Specifies the compilation mode. `release` is the default value, and `debug` indicates the debugging mode.

---

## Installation

### Executing the Installation Script

1. As the `root` user, navigate to the `oGRAC/build` directory.
2. Use the installation script for deployment, and specify the installation user or compatibility.

```bash
sh local_install.sh install -u [user_name]

# sh local_install.sh install -u [user_name] -c A  # Create a database with compatibility A.
```

The script automatically performs the following tasks:

* Verify the runtime environment and user permissions.
* Create the installation user and home directory (if they do not exist).
* Stop and clean up residual historical processes and data.
* Create the data directory and log directory.

After the execution is complete, you can log in as the user specified by `-u` and run the `ogsql / as sysdba` command to connect to the database.

If a restart is required, run the following command to start the database:

```shell
/home/[user_name]/install/bin/ogracd -D /home/[user_name]/data &
```

---

### Uninstallation and Cleanup

If redeployment or environment cleanup is required, execute the uninstallation script:

```bash
sh local_install.sh clean -u [user_name]
```

This operation stops the service and removes the data directory, installation directory, and related environment variables.

---

## Debug and Debugging Suggestions

In a single-node deployment, the system still includes the Cluster Manager (CM) component by default. When using `gdb` for breakpoint debugging, CM may misjudge the database as abnormal due to heartbeat timeout and forcibly terminate the process.

To avoid this issue, it is recommended to adjust the CM timeout parameters before debugging:

```bash
su - [user_name]
cms res -edit db -attr HB_TIMEOUT=100000000
cms res -edit db -attr CHECK_TIMEOUT=10000000
```

> [!NOTE]
>
> * The above configuration is recommended for debugging environments only.
> * After debugging is complete, the default configuration can be restored to avoid affecting system behavior.

---

## Summary

At this point, the oGRAC single-node local environment has completed the full process from source code acquisition, compilation, and installation to debugging configuration. Subsequent parameter tuning, functional verification, or source-level debugging can be performed based on specific requirements.

If a multi-node or production environment is required, refer to the two-node cluster deployment documentation.
