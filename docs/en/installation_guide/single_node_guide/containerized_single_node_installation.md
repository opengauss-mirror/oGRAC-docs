# oGRAC Single-Node Containerized Installation

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:48:30.801Z pushedAt=2026-08-17T00:46:29.679Z -->

Note that a single-node deployment is intended only for personal development and verification, and is not recommended for production environments. This document provides a Docker-based installation method, with the operating system being openEuler 22.03 LTS.

## 1. Downloading the Docker Image

```shell
wget https://repo.openeuler.org/openEuler-22.03-LTS/docker_img/aarch64/openEuler-docker.aarch64.tar.xz

docker load < ./openEuler-docker.aarch64.tar.xz
```

## 2. Viewing the Image File

As the root user, enter:

```shell
docker images
```

Under normal circumstances, the following information is displayed:

```shell
REPOSITORY            TAG        IMAGE ID        CREATED                 SIZE
openeuler-22.03-lts   lastest    xxxx            About a minute ago      3.71GB
```

## 3. Creating and Starting Docker

```shell
docker run --name mirror_name -itd -v /home/user_name/docker/data:/home --privileged=true --network=host --shm-size=128g IMAGE_ID tail -f /dev/null
```

- `mirror_name` is the Docker container name, which can be customized.
- `-v` specifies the Docker mount, which mounts the `/home/user_name/docker/data` directory on the host machine to the `/home` directory inside the container.
- `--shm-size` specifies the Docker shared memory size, which is set to `128g` here. It is recommended that it not be less than `128g`.
- `IMAGE_ID` is the ID of the Docker image, which can be viewed using `docker images`.

## 4. Viewing and Accessing the Docker Container

4.1 View the Docker container.

```shell
docker ps
```

Under normal circumstances, the following information is displayed:

```shell
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS               NAMES
f862ea8687aa        70802bc91797        "tail -f /dev/null"   33 minutes ago      Up 33 minutes                           mirror_name
```

4.2 Access the Docker container.

```shell
docker exec -it mirror_name /bin/bash
```

## 5. Configuration Inside the Docker Image

HTTP and Git proxies need to be configured (not required for non-compilation-based installation). Detailed instructions are not provided here. Refer to the official Git website or other resources for more information.

5.1 To perform a compilation-based installation, install the following dependencies:

```shell
yum install -y libaio-devel openssl openssl-devel ndctl-devel \
ncurses ncurses-devel libtirpc-devel expect ant bison iputils \
iproute wget make gcc gcc-c++ gdb gdb-gdbserver python3 python3-devel \
git net-tools cmake automake byacc libtool git unzip vim lz4 lz4-devel patch \
xz flex unixODBC-devel unixODBC --skip-broken
```

Requirements:
The cmake version must be 3.12 or later.
The lz4 version must be 1.8.3 or later.

5.2 To perform a package-based installation, install the following dependencies:

```bash
yum install -y wget python3 python3-devel iputils iproute --skip-broken
```

## 6. Compiling oGRAC

The preceding steps have completed the process from creating the image to entering the container. Next, compile and install oGRAC inside the container.

6.1 Download files.

Download URL:

```shell
git clone https://gitcode.com/opengauss/oGRAC.git
```

6.2 Modify `Makefile.sh`.

Navigate to the `oGRAC/build` directory and execute the following command to replace `USE_PROTECT_VM=ON` with `USE_PROTECT_VM=OFF` in the file:

```shell
sed -i 's+USE_PROTECT_VM=ON+USE_PROTECT_VM=OFF+' Makefile.sh
```

6.3 Compile and install oGRAC.

Navigate to the `oGRAC/build` directory and execute the following command to compile and install. This example compiles the debug version. If `-b` is not specified, the release version is compiled by default. `-u` specifies the installation user name. `-c` specifies the compatibility, which supports A, B, or C compatibility; if not specified, compatibility A is used by default.

```shell
sh local_install.sh prepare

sh local_install.sh compile -b debug

sh local_install.sh install -u user_name

# sh local_install.sh install -u [user_name] -c A  # Create a database with compatibility A.
```

At this point, oGRAC has been compiled and installed in the container, and it can be configured and used as needed.

## 7. Additional Notes

In addition to compiling and installing oGRAC from source as described above, you can also install it using an installation package within the Docker container.

You can obtain the installation package from the **Download** page on the [openGauss official website](https://docs.opengauss.org/en/).

After downloading the installation package, extract it using tar, navigate to the `oGRAC/build` directory, and execute the following command:

```shell
sh local_install.sh install -u user_name
```
