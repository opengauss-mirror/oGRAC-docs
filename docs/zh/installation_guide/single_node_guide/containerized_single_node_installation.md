# 容器化单节点安装 oGRAC

请注意，单节点仅用于个人开发验证，不建议用于生产环境。本文档提供一种基于 Docker 的安装方式，操作系统为 openEuler 22.03 LTS。

## 1. 下载 docker 镜像

```shell
wget https://repo.openeuler.org/openEuler-22.03-LTS/docker_img/aarch64/openEuler-docker.aarch64.tar.xz

docker load < ./openEuler-docker.aarch64.tar.xz
```

## 2. 查看镜像文件

在 root 用户下输入：

```shell
docker images
```

正常情况下会回显如下信息：

```shell
REPOSITORY            TAG        IMAGE ID        CREATED                 SIZE
openeuler-22.03-lts   lastest    xxxx            About a minute ago      3.71GB
```

## 3. 创建并启动 docker

```shell
docker run --name mirror_name -itd -v /home/user_name/docker/data:/home --privileged=true --network=host --shm-size=128g IMAGE_ID tail -f /dev/null
```

- mirror_name 是 docker 的容器名，可以自定义
- -v 是 docker 的挂载，将宿主机的 `/home/user_name/docker/data` 目录挂载到容器内的 `/home` 目录下
- --shm-size 是 docker 的共享内存大小，这里设置为 128g，建议不要小于 128g
- IMAGE_ID 是 docker 镜像的 ID，可以通过 `docker images` 查看

## 4. 查看并进入 docker

4.1 查看 docker

```shell
docker ps
```

正常情况下会回显如下信息：

```shell
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS               NAMES
f862ea8687aa        70802bc91797        "tail -f /dev/null"   33 minutes ago      Up 33 minutes                           mirror_name
```

4.2 进入 docker

```shell
docker exec -it mirror_name /bin/bash
```

## 5. Docker 镜像内配置

需要配置 http 代理、git 代理（非编译安装不需要），此处不做详细说明，详情参考 git 官网等。

5.1 若为编译安装则还需安装依赖：

```shell
yum install -y libaio-devel openssl openssl-devel ndctl-devel \
ncurses ncurses-devel libtirpc-devel expect ant bison iputils \
iproute wget make gcc gcc-c++ gdb gdb-gdbserver python3 python3-devel \
git net-tools cmake automake byacc libtool git unzip vim lz4 lz4-devel patch \
xz flex unixODBC-devel unixODBC --skip-broken
```

要求：
cmake 版本在 3.12 及以上；
lz4 版本在 1.8.3 及以上。

5.2 若为安装包安装则还需安装依赖：

```bash
yum install -y wget python3 python3-devel iputils iproute --skip-broken
```

## 6. 编译 oGRAC

经过上述步骤已完成从创建镜像到进入容器的过程，接下来在容器内编译安装 oGRAC。

6.1 下载文件

下载地址为

```shell
git clone https://gitcode.com/opengauss/oGRAC.git
```

6.2 修改 Makefile.sh

进入 `oGRAC/build` 目录下，执行以下命令替换文件里的 `USE_PROTECT_VM=ON` 为 `USE_PROTECT_VM=OFF`

```shell
sed -i 's+USE_PROTECT_VM=ON+USE_PROTECT_VM=OFF+' Makefile.sh
```

6.3 编译安装 oGRAC

进入 `oGRAC/build` 目录下，执行以下命令进行编译安装。示例为编译 debug 版本，不指定 `-b` 时默认编译 release 版本；`-u` 指定安装用户名；`-c` 指定兼容性，支持指定 A/B/C 兼容性，不指定时默认为 A 兼容性

```shell
sh local_install.sh prepare

sh local_install.sh compile -b debug

sh local_install.sh install -u user_name

# sh local_install.sh install -u [user_name] -c A  # 新建兼容性为A的数据库
```

至此容器内已编译安装好 oGRAC，后续可以根据需要进行配置和使用。

## 7. 其他补充

在 docker 容器内除上述编译安装 oGRAC 外，还可以使用安装包进行安装。

可以在 [openGauss 官网](https://docs.opengauss.org/zh/)的 `下载` 页面获取安装包。

下载安装包后，使用 tar 解压安装包，然后进入 `oGRAC/build` 目录下，执行如下命令即可：

```shell
sh local_install.sh install -u user_name
```