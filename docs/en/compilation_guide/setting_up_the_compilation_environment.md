# Setting Up the Compilation Environment

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:46:33.742Z pushedAt=2026-08-17T00:46:29.632Z -->

## Initializing the System

Disable SELinux and the firewall:

```shell
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

## Creating Directories and Users

```shell
mkdir -p compile_path
chmod 755 -R compile_path
useradd user_name
passwd user_password
chown -R user_name:user_name compile_path
```

## Installing Required Dependencies

```shell
yum install -y libaio-devel openssl openssl-devel ndctl-devel perl \
ncurses ncurses-devel libtirpc-devel expect ant bison iputils rpm-build \
iproute wget make gcc gcc-c++ gdb gdb-gdbserver python3 python3-devel \
git net-tools cmake automake byacc libtool lz4-devel patch \
--skip-broken unixODBC unixODBC-devel
```

Requirements:
cmake version 3.12 or later;
lz4 version 1.8.3 or later.
