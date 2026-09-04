# Compilation and Installation

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:48:25.826Z pushedAt=2026-08-17T00:46:29.677Z -->

## Environment Setup

### System Initialization

Disable SELinux and the firewall.

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

### Creating Directories and Users

```bash
mkdir -p [compile_path]
chmod 755 -R [compile_path]
useradd [user_name]
passwd [user_password]
chown -R [user_name]:[user_name] [compile_path]
```

### Installing Required Dependencies

```bash
yum install -y libaio-devel openssl openssl-devel ndctl-devel \
ncurses ncurses-devel libtirpc-devel expect ant bison iputils \
iproute wget make gcc gcc-c++ gdb gdb-gdbserver python3 python3-devel \
git net-tools cmake automake byacc libtool --skip-broken unixODBC unixODBC-devel
```

---

## Obtaining and Compiling Source Code

### Obtaining Source Code

```bash
cd [compile_path]
git clone https://gitcode.com/opengauss/oGRAC.git
```

### Modifying the Build Configuration

If you need to disable the virtual memory protection option (recommended when building and installing a debug version).

```bash
cd oGRAC/build
sed -i 's/DUSE_PROTECT_VM=ON/DUSE_PROTECT_VM=OFF/g' Makefile.sh
```

---

## Compilation and Installation Process

### Preparing the Compilation Environment

Install all dependencies required for compiling and running oGRAC.

```bash
sh local_install.sh prepare
```

### Compilation

```bash
sh local_install.sh compile -b debug
```

- `-b, --build_type=<type>`: Specifies the build type (`release`/`debug`, defaults to `release`).
- `-d, --deps=no`: Do not compile dependencies.

Build output directory: `oGRAC/oGRAC-DATABASE-*-64bit`

### Installation

```bash
sh local_install.sh install -u [user_name]
```

- `-u, --user=<user>`: Specifies the system user for installation and operation, defaulting to `ogracdba`.

The installation process includes:

- Creating the user and home directory (if they do not exist)
- Stopping old processes and cleaning up old data
- Creating the data directory
- Running `install.py` for initialization

Parameters in `install.py`:

| Parameter                 | Default Value                        | Description                       |
| ---------------------| ------------------------------------ | -------------------------- |
| -U \<user:group>      | ogracdba:ogracdba                    | System user and group for installation and operation       |
| -R <install_path>    | /home/ogracdba/install             | Software installation path               |
| -D <data_path>       | /home/ogracdba/data                | Data file storage path           |
| -l <log_path>        | /home/ogracdba/logs/install.log    | Installation log file path           |
| -M \<mode>            | ogracd_in_cluster                    | Running mode. `ogracd`: standalone mode; `ogracd_in_cluster`: cluster mode  |
| -N 0                 | 0                                    | Node ID, which can only be `0` or `1`        |
| -W \<IP>              | 192.168.0.1                          | Configure the database IP allowlist         |
| -g withoutroot       | withoutroot                          | Indicates that the installation script is allowed to run without root privileges, but the installation directory must have proper permissions. The parameter value can only be `withoutroot`. To use it, you must specify `-g withoutroot` exactly.         |
| -d                   | No parameter value                              | Indicates running in the background                   |
| -c                   | No parameter value                              | Indicates not using SSL-based secure connections, with automatic related configuration                   |
| -Z _SYS_PASSWORD=... | _SYS_PASSWORD=huawei@1234            | Default password for the administrator account. It is recommended to modify the configuration.   |
| -Z SESSIONS=...     | SESSIONS=1000                        | Maximum number of sessions                 |

After the above steps, oGRAC has been compiled and installed. You can proceed with configuration and usage as needed.

### Uninstallation and Cleanup

Stop the service and delete data, installation directories, and related environment variables.

```bash
sh local_install.sh clean -u [user_name]
```
