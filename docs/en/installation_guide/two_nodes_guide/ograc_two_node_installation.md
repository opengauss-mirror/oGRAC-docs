# oGRAC Two-Node Deployment

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:49:34.712Z pushedAt=2026-08-17T00:46:29.696Z -->

## 1. Document Introduction

This document is intended to guide developers through the **two-node cluster installation and deployment** of oGRAC on **two physical machines or virtual machines**.

---

## 2. Prerequisites

### 2.1 Hardware Requirements

A minimum of two servers is required for oGRAC two-node deployment. The recommended hardware specifications are as follows:

* Number of hosts: 2 Arm-based physical machines or DCS virtual machines
* Minimum recommended configuration per host:
  * Memory: 16 GB
  * CPU: 8 cores
  * Available disk space: no less than 100 GB
* Shared disk requirements:
  * At least 4 raw LUN disks are required and they must not be partitioned LUNs. The storage node where they reside must be on the same network as both hosts and be directly accessible.

> **NOTE**: Insufficient resources may cause installation failures, especially during the initialization of shared storage and CM components.

---

### 2.2 Operating System Requirements

* Supported operating system versions:

  * openEuler 20.03 LTS (AArch64)
  * openEuler 22.03 LTS (AArch64)
  * openEuler 24.03 LTS (AArch64)

> **Recommendation**: It is recommended to use the installation packages for the above versions. For other system environments, you may compile the installation packages yourself, but official full compatibility verification has not been conducted.

---

## 3. Installation Preparations

The following steps must be performed on both nodes separately, using the `root` user unless otherwise specified.

### 3.1 System Initialization

> **Tip**: The following operations are recommended only for testing or non-production environments. If they need to be performed in a production environment, consult operations management personnel to confirm security policies first, and do not disable the firewall directly.

Disable SELinux and the firewall to prevent them from affecting node communication and database process startup.

```shell
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

### 3.2 Installing System Dependencies

oGRAC installation depends on Python, time synchronization, and network tools. Run the following commands on both nodes:

```shell
yum install -y wget ntpdate chrony python3 python3-devel iputils iproute patchelf lz4 --skip-broken
```

The lz4 version must be 1.8.3 or later.

Note: The official repository of openEuler 20.03 does not include the `patchelf` installation package. If you are using openEuler 20.03, you need to manually install `patchelf`:

```shell
yum install -y gcc make automake autoconf libtool
# Clone patchelf. Version 0.18.0 is recommended.
git clone https://github.com/NixOS/patchelf.git
cd patchelf
./bootstrap.sh
# You can use --prefix to specify the installation path based on your needs.
./configure --prefix=/usr/local
make -j$(nproc)
make install
# The output should be similar to patchelf 0.18.0.
patchelf --version
```

---

## 4. Installation Package Download

### 4.1 Creating the Installation Directory

It is recommended that you create a separate installation directory (this document uses the `/data` directory as an example). Do not download or install software packages in `/home` or system directories to avoid permission issues.

```shell
mkdir -p /data/ograc
cd /data/ograc
```

---

### 4.2 Downloading the Installation Package

The following command can be used to check the operating system version of the node, so as to obtain the installation package for the corresponding architecture:

```shell
cat /etc/os-release
```

In the current directory of node 0 and node 1, download the corresponding official installation package from the official website. The download URLs for the installation packages of each Arm system version are as follows:

```shell
# openEuler 20.03
wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/7.0.0-RC3/oGRAC/openGauss-oGRAC-openEuler20.03-aarch64-RELEASE.tgz
# openEuler 22.03
wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/7.0.0-RC3/oGRAC/openGauss-oGRAC-openEuler22.03-aarch64-RELEASE.tgz
# openEuler 24.03
wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/7.0.0-RC3/oGRAC/openGauss-oGRAC-openEuler24.03-aarch64-RELEASE.tgz
```

---

## 5. Two-Node Shared Storage Preparation

### 5.1 Planning LUNs

The oGRAC two-node cluster requires shared storage. Prepare **4 LUNs** on the storage side in advance and complete host group mapping so that both nodes can access these four disks.

The recommended planning is as follows (example):

* 1 × 5 GB: CM quorum disk
* 1 × 4 TB: redo disk
* 2 × 2 TB: data disk and archive disk

The LUN capacity can be adjusted appropriately based on actual service requirements. For performance-sensitive environments, ensure that the data disk and redo disk have sufficient capacity to meet log and service data demands. It is recommended that the disk sizes satisfy the following: `data disk > redo disk > archive disk > CM quorum disk`.

Query the four allocated disks using the following command to obtain the device identifiers starting with `scsi` or `wwn`:

```shell
ll /dev/disk/by-id
```

Then link the four disks to the following directories:

```shell
ln -s /dev/disk/by-id/scsi-disk1 /dev/dss-disk1 # Data disk
ln -s /dev/disk/by-id/scsi-disk2 /dev/dss-disk2 # Redo disk
ln -s /dev/disk/by-id/scsi-disk3 /dev/dss-disk3 # Archive disk
ln -s /dev/disk/by-id/scsi-disk4 /dev/gcc-disk # CM quorum disk
```

The purposes of the four disks are described in the following table:

| Symbolic Link | Purpose     | DSS Volume   | Recommended Size |
| ------------- | ----------- | ------------ | ----------------- |
| gcc-disk      | CM quorum disk | Not managed by the LUN management component | 5 GB                |
| dss-disk1     | Data disk    | vg1          | 2 TB                |
| dss-disk2     | Redo disk    | vg2          | 4 TB                |
| dss-disk3     | Archive disk | vg3          | 2 TB                |

> **NOTE**: `gcc` is the internal name of the CM component and is unrelated to the compiler.

Authorize these four disks so that the `DSS` component can access them. Run the following command:

```shell
ll /dev/disk/by-id/scsi-disk1
# Obtain the target disk device that the /dev/sdx symbolic link points to.
chmod 777 /dev/sdx
```

Then proceed with the subsequent installation steps.

---

### 5.2 Extracting the Installation File

Execute the following on both nodes, where `[os version]` indicates the architecture version:

```shell
cd /data/ograc
tar -zxvf openGauss-oGRAC-openEuler[os version]-aarch64-RELEASE.tgz
chmod -R 777 ograc_connector
chown -R root:root ograc_connector
```

---

### 5.3 Configuring Time Synchronization

The cluster environment requires high time consistency. First run the `date` command to check whether the time on each node is consistent. If they are consistent, you may skip this step. Otherwise, you must complete time synchronization. (If virtual machines are used, disable the time synchronization policy with the host machine first to prevent time jumps.)

#### Case 1: Nodes Can Access the External Network

Execute the following command on both nodes respectively:

```shell
ntpdate -u [External NTP server URL]
```

#### Scenario 2: No External Network

* Node 0 serves as the time server.
* Node 1 synchronizes time with node 0.

**Node 0:**

```shell
sed -i "1i allow all" /etc/chrony.conf
systemctl restart chronyd
sed -i 's/^#local stratum 10/local stratum 10/' /etc/chrony.conf
ss -unlp | grep chronyd
```

**Node 1:**

```shell
sed -i "1i server [Node 0 IP address] iburst" /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd  # If the time deviation becomes too large later due to other factors, this command can be used to quickly trigger forced synchronization.
chronyc tracking
```

---

## 6. Installation Parameter Configuration

### 6.1 Modifying the Configuration File

Go to the `action` directory and edit `config_params_lun.json`:

```shell
cd /data/ograc/ograc_connector/action
vim config_params_lun.json
```

Key configuration notes:

1. The `node_id` values for both nodes must be `0` and `1`, respectively.
2. For machines with smaller memory (such as DCS virtual machines or physical machines with less than 300 GB of memory), set `auto_tune = 1`.
3. `redo_num × redo_size × 2` should be less than the redo disk size.

Example for node 0 (for node 1, only `node_id` needs to be changed to `1`):

```json
{
    "deploy_mode": "dss",
    "node_id": "0",
    "cms_ip": "xxx.xxx.xxx.xxx;xxx.xxx.xxx.xxx",
    "db_type": "1",
    "mes_ssl_switch": false,
    "MAX_ARCH_FILES_SIZE": "300G",
    "redo_num": "6",
    "redo_size": "5G",
    "auto_tune": "1",
    "dss_vg_list": {
        "vg1": "/dev/dss-disk1",
        "vg2": "/dev/dss-disk2",
        "vg3": "/dev/dss-disk3"
    },
    "gcc_home": "/dev/gcc-disk",
    "cms_port": "14587",
    "dss_port": "1811",
    "ograc_port": "1611",
    "interconnect_port": "1601,1602",
    "_SHM_KEY": 17,
    "module_config": {
        "ograc_home": "/data/ograc_install/ograc",
        "data_root": "/data/ograc_install/dbdata",
        "user": "ograc"
    }
}
```

The fields are described as follows:

* `deploy_mode`: Installation mode. The `dss` mode should be used for the current installation.
* `node_id`: Node ID, starting from `0`.
* `cms_ip`: When the service network and heartbeat network are not separated, fill in the IP addresses of both node hosts.
* `db_type`: Database identifier. Modification is not recommended.
* `mes_ssl_switch`: Indicates whether MES communication is encrypted via SSL.
* `MAX_ARCH_FILES_SIZE`: Maximum archive file size. It is recommended not to exceed the archive disk size.
* `redo_num`: Number of redo files.
* `redo_size`: Redo file size. Since the first database startup will erase all content on the disk using `dd`, it is not recommended to set an excessively large value, as this may result in a prolonged first-time startup.
* `auto_tune`: Indicates whether to enable adaptive parameter configuration (recommended for small-spec machines).
* `dss_vg_list`: Corresponds to the data disk, redo disk, and archive disk devices, respectively.
* `gcc_home`: CM quorum disk device.
* `cms_port`: CMS port, customizable.
* `dss_port`: DSS port, customizable.
* `ograc_port`: oGRAC database port, customizable.
* `interconnect_port`: inter-node communication port for the oGRAC database, customizable.
* `_SHM_KEY`: Shared memory key value. If multiple databases need to be installed in the current environment, it is recommended to modify this to a unique key value to avoid conflicts.
* `module_config`: Module installation configuration. `ograc_home` is the oGRAC installation directory, `data_root` is the database data directory (it is recommended to use the same root directory as the installation directory), and `user` is the database user. All of these are customizable.

---

### (Optional) 6.2 Modifying Database Compatibility

```shell
cd /data/ograc/ograc_connector/action
vim ograc/install_config.json
```

Add the `DBCOMPATIBILITY` field to declare the database compatibility, which supports `A`/`B`/`C`, as shown below:

```json
{
  "DBCOMPATIBILITY": "A"
}
```

If this is the first installation of the database using the current package, the `install_config.json` file must be created in advance. If the database has already been installed, simply modify the corresponding `DBCOMPATIBILITY` field.

## 7. Cluster Installation and Startup

For common issues encountered during installation and deployment, see the [oGRAC Installation and Deployment Troubleshooting](./installation_deployment_issues.md) section.

### 7.1 Installing Nodes

Perform the following steps on both nodes sequentially. It is recommended to wait until the installation on node 0 is complete before proceeding with node 1. First, perform the pre-installation:

```shell
sh appctl.sh pre_install config_params_lun.json
```

After the pre-installation succeeds, perform the installation:

```shell
sh appctl.sh install config_params_lun.json
```

During each installation, set the password of the  `sys` user when prompted:

```shell
please enter ograc_sys_pwd:
```

The password must be a combination of letters, digits, and special characters. Uppercase letters are not required. The password set on both nodes must be identical.

---

### 7.2 Starting Nodes

It is recommended to start node 0 first, then node 1, and execute the following commands in sequence on both nodes:

```shell
sh appctl.sh start
```

Note that the first `start` on node 0 creates redo and data files, which takes a relatively long time. Please wait patiently. The first `start` on node 1 does not involve this process and takes relatively less time.

---

## 8. Cluster Status Check

Run the following command on any node to query the cluster status:

```shell
su -s /bin/bash ograc
cms stat -res db
```

See the following figure:

![Cluster status query](image-cluster-status.png)

Pay attention to the `STAT` column, which indicates the current node status. When both nodes are `ONLINE`, the cluster node status is normal.

`PRE_STAT` is the previous node status and `TARGET_STAT` is the ideal status, which can serve as a reference status in failure scenarios.

At this point, the database functionality can be briefly verified on the cluster by executing the following on both nodes respectively:

```shell
su -s /bin/bash ograc
ogsql / as sysdba -q
```

Access the database command terminal and execute the following on one of the nodes:

```shell
create table test(a int);
insert into test values(123);
commit;
```

Then execute the following on the other node:

```shell
select * from test;
```

The previously inserted data can then be obtained, indicating that the database functionality is normal.

---

### (Optional) Running the oGRAC Two-Node Multi-Write Test Demo

After the cluster is successfully installed, refer to [oGRAC Two-Node Multi-Write Test Demo](./ograc_two_nodes_multiwrite_testdemo.md) to run the oGRAC two-node multi-write test demo.

## 9. oGRAC Reinstallation

If oGRAC needs to be redeployed in this environment, the cluster must first be stopped, uninstalled, and cleaned up before reinstallation can proceed.

### 9.1 Stopping Services

On node 0 and node 1, navigate to the `/data/ograc/ograc_connector/action` directory and execute the following command as the `root` user to stop the nodes:

```shell
cd /data/ograc/ograc_connector/action
sh appctl.sh stop
```

### 9.2 Uninstallation and Cleanup

Run the following command on node 0 and node 1 respectively to perform the uninstallation:

```shell
sh appctl.sh uninstall override
```

After that, you can switch to a different package version or modify the configuration file, and then perform reinstallation by following the steps in this document.

---

## 10. Summary

This concludes the oGRAC two-node deployment guide. If you have any questions, please contact the openGauss official community, where relevant developers will provide support.
