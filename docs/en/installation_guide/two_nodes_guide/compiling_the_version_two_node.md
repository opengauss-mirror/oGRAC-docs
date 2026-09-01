# oGRAC Compilation and Packaging

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:48:52.241Z pushedAt=2026-08-17T00:46:29.685Z -->

## About This Document

This document is intended to guide developers through the **source code compilation and build package generation for oGRAC in a local environment, for use in a two-node installation**.

> **Application Scope**
>
> * Supports compilation of both `release` and `debug` versions.
> * Uses the Arm architecture (openEuler) environment as the default example.
> * Applicable to development, testing, and version verification scenarios.

> [!WARNING]
>
> * If the local environment is openEuler 24.03 LTS (AArch64), the compilation and packaging steps may differ slightly. You may still refer to this document for the specific procedures, but pay attention to the content highlighted in red.

---

## Compilation Environment Setup

### System Initialization

Before starting compilation, you need to disable system security policies that may affect compilation or runtime. Run the following commands as the **root** user:

```shell
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

> [!NOTE]
>
> * SELinux and firewall may block local debugging or component communication.
> * It is recommended to perform the above operation only in development or intranet environments.

---

### Creating the Compilation Directory and User

To ensure permission isolation and environment cleanliness, it is recommended to use an **independent system user** for source code compilation and package generation operations.

```shell
mkdir -p [compile_path]
chmod 755 -R [compile_path]
useradd [user_name]
passwd [user_name]
chown -R [user_name]:[user_name] [compile_path]
```

> [!NOTE]
>
> * `[compile_path]`: The unified directory for oGRAC source code, third-party libraries, and compilation artifacts.
> * `[user_name]`: The system user dedicated to compiling oGRAC.

---

### Installing System Dependencies

oGRAC compilation depends on many system libraries and build tools. Run the following command on the compilation host to install them all at once:

```shell
yum install -y libaio-devel openssl openssl-devel ndctl-devel unixODBC-devel unixODBC \
ncurses ncurses-devel libtirpc-devel expect ant bison iputils  \
iproute wget make gcc gcc-c++ gdb gdb-gdbserver python3 python3-devel \
git net-tools cmake automake byacc libtool lz4 lz4-devel patch xz flex --skip-broken
```

> [!NOTE]
>
> * The above dependencies cover components required for database compilation, debugging, and installation script execution.
> * `cmake`: version >= 3.12 is required.
> * `lz4`: version >= 1.8.3 is required.
> * If dependency conflicts exist, `--skip-broken` can prevent installation interruption.

> [!WARNING]
>
> <span style="color: red;">On openEuler 24.03 LTS, do not install CMake via `yum install`, as the version provided is too high and may result in conflicts. It is recommended to compile CMake using GCC 10.3.0 or 10.3.1. The compilation steps are as follows:</span>
>
> 1. Switch to the sub-user and go to the directory for compiling CMake. Here, `/home/user_name` is used as an example.
>
>    ```shell
>    $ su user_name
>    $ cd /home/user_name
>    ```
>
> 2. Download GCC. This example uses the one from the openGauss-third_party repository (the recommended GCC version is 10.3.0/10.3.1):
>
>    ```shell
>    $ wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2403_arm.tar.gz
>    ```
>
> 3. Decompress the third-party library package to obtain the GCC binary path. The example path here is `/home/user_name/openGauss-third_party_binarylibs_openEuler_2403_arm/buildtools/gcc10.3/gcc/lib`.
>
>    ```shell
>    $ tar -zxvf openGauss-third_party_binarylibs_openEuler_2403_arm.tar.gz
>    ```
>
> 4. Download the CMake source package. This example uses the official CMake version 3.22.1:
>
>    ```shell
>    $ wget https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1.tar.gz
>    ```
>
> 5. Decompress the CMake source package, go to the CMake source directory, and compile CMake using GCC 10.3:
>
>    ```shell
>    # Decompress.
>    $ tar -zxvf cmake-3.22.1.tar.gz
>    $ cd cmake-3.22.1
>
>    # Configure the compiler.
>    $ export CC=/home/user_name/openGauss-third_party_binarylibs_openEuler_2403_arm/buildtools/gcc10.3/gcc/bin/gcc
>
>    $ export CXX=/home/user_name/openGauss-third_party_binarylibs_openEuler_2403_arm/buildtools/gcc10.3/gcc/bin/g++
>
>    $ export LD_LIBRARY_PATH=/home/user_name/openGauss-third_party_binarylibs_openEuler_2403_arm/buildtools/gcc10.3/isl/lib:$LD_LIBRARY_PATH
>
>    $ export LD_LIBRARY_PATH=/home/user_name/openGauss-third_party_binarylibs_openEuler_2403_arm/buildtools/gcc10.3/mpfr/lib:$LD_LIBRARY_PATH
>
>    # Configure.
>    $ ./bootstrap --prefix=/home/user_name/cmake-3.22.1 --parallel=$(nproc) --no-system-libs
>
>    # Compile and install.
>    $ make -j $(nproc)
>    $ make install
>
>    # Verify.
>    $ /home/user_name/cmake-3.22.1/bin/cmake --version
>    ```
>
> 6. If a version number is echoed after verification, the compilation and installation have completed successfully. The CMake binary files will be generated in the `/home/user_name/cmake-3.22.1` directory. The following configuration is also required before proceeding with subsequent operations:
>
>    ```shell
>    $ export PATH=/home/user_name/cmake-3.22.1/bin:$PATH
>    $ export LD_LIBRARY_PATH=/home/user_name/cmake-3.22.1/lib:$LD_LIBRARY_PATH
>    $ export CMAKEROOT=/home/user_name/cmake-3.22.1
>    ```

---

## Compilation

After the compilation environment is prepared, you can proceed with oGRAC version compilation and package generation.

### Downloading and Preparing Third-Party Dependency Libraries

oGRAC depends on the third-party binary libraries provided by openGauss. Download and extract them to the oGRAC source code directory first.

Note that the current mainline branch contains two branches: `master` and `1.0.0`. The official documentation uses `1.0.0` as the baseline for installation and deployment guidance.

```bash
su - [user_name]
cd [compile_path]

git clone https://gitcode.com/opengauss/oGRAC.git

cd oGRAC
# Switch to the desired branch as needed.
# git reset --hard origin/1.0.0
wget --no-check-certificate \
https://opengauss.obs.cn-south-1.myhuaweicloud.com/6.0.0/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz

tar -zxf openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz
```

> [!WARNING]
>
> * Ensure that the third-party library extraction directory is at the same level as the source code directory.
> * It is not recommended to arbitrarily modify the third-party library directory name.

---

### Executing the Build Script

Go to the `build` directory and generate the installation package using the unified build script.

> [!NOTE]
>
> * For the debug version, disable the virtual memory protection option to avoid database startup failure caused by VM page configuration issues in certain environments. Run the following:
>
>   ```bash
>   $ cd [compile_path]/oGRAC/build
>   $ sed -i 's/DUSE_PROTECT_VM=ON/DUSE_PROTECT_VM=OFF/g' Makefile.sh
>   ```

```bash
cd [compile_path]/oGRAC/build
# Basic usage (third-party libraries extracted in the oGRAC directory)
sh build_ograc.sh [release|debug] --with-dss

# Specify the directory containing third-party libraries (the path must be within the oGRAC directory, pointing to the parent directory that contains the third-party library folder)
sh build_ograc.sh [release|debug] --with-dss --third-party-path <path>
```

Parameter description:

| Parameter | Description |
|------|------|
| `release` | Compiles the release version, suitable for feature verification and delivery. |
| `debug` | Compiles the debug version, which includes debug symbols for gdb debugging. |
| `--with-dss` | Enables DSS-related components (mandatory for two-node or higher deployments). |
| `--third-party-path <path>` | Specifies the search path for third-party dependencies, pointing to the parent directory that contains the `openGauss-third_party_binarylibs_*` folder. **The path must be within the `oGRAC` directory.** If not specified, the script searches the oGRAC root directory by default. |
| `-h`, `--help` | Displays help information. |

> **Pre-compilation Environment Check**
>
> The build script has a built-in environment pre-check feature that automatically checks third-party library dependencies, OS version compatibility, and other items before compilation. If the third-party libraries have been extracted into the `oGRAC` directory, the script will locate them automatically without additional specification.
>
> To verify whether the environment configuration is correct in advance, you can also run the environment check separately (without performing compilation):
> 
> ```bash
> cd [compile_path]/oGRAC/build
> sh build_dss.sh [release|debug] --check-only
> ```
>
> If the third-party libraries are not in the default path, you can specify the path (optional):
> 
> ```bash
> sh build_dss.sh [release|debug] <path> --check-only
> ```
>
> `--check-only` performs only the pre-compilation environment check without executing the actual compilation process.

> [!NOTE]
>
> * The compilation process takes a long time. Please wait patiently.
> * If compilation fails, check whether the dependencies are complete first.

---

## Installation Package Output

After the compilation is complete, the generated installation package will be located in the following directory:

```shell
[compile_path]/oGRAC/package/
```

This directory contains the oGRAC installation package (in `.tar.gz` format). Developers can distribute this installation package to target nodes and deploy it according to the corresponding installation documentation.

---

## Summary

At this point, the source code compilation and packaging process for oGRAC is complete. Before using the installation package, it is recommended to confirm whether its corresponding compilation type (`release`/`debug`) matches the target deployment environment.

For debugging deployment or cluster installation, refer to the corresponding installation and deployment documentation.
