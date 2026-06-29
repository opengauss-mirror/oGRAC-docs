# 因系统 lz4 版本低于 1.8.3 导致编译失败的问题

## 一、问题现象

执行 `sh local_install.sh compile -b debug` 或 `sh build_ograc.sh debug --with-dss` 编译 oGRAC 时，编译失败。会出现如下类似报错信息:

```text
error: 'LZ$F_INIT_PREFERENCES' undeclared (first use in this function)；did you mean 'OG_PRIV_REFERENCES'?
```

## 二、定位方法

1. 查看编译日志，确认报错是否与 lz4 相关。
2. 检查当前系统已安装的 lz4 版本。

   ```bash
   lz4 --version
   # 或
   rpm -qa | grep lz4
   ```

3. 检查 lz4 头文件是否存在。

   ```bash
   ls /usr/include/lz4*.h
   ```

## 三、问题根因

oGRAC 编译依赖 lz4 压缩库，且在备份/压缩等模块中使用了 lz4 的 frame API，例如 `LZ4F_INIT_PREFERENCES` 等宏和函数。这些 frame API 是在 lz4 1.8.3 及更高版本中才引入的。

当前编译报错 `LZ4F_INIT_PREFERENCES undeclared`，说明编译器在当前系统的 lz4 头文件中没有找到该宏定义，即系统安装的 lz4 版本低于 1.8.3。由于版本不兼容，oGRAC 编译时无法正确解析相关接口，从而导致编译失败。

## 四、解决方法

1. 安装或升级 lz4 及其开发包。

   ```bash
   yum install -y lz4 lz4-devel
   ```

2. 安装完成后，验证 lz4 版本是否满足要求。

   ```bash
   lz4 --version
   ```

   期望输出中包含版本号，且版本号大于等于 1.8.3，例如：

   ```text
   *** LZ4 command line interface 64-bits v1.9.2, by Yann Collet ***
   ```

3. 如果 lz4 版本满足要求的话，重新编译 oGRAC 即可。

4. 如果 yum 源中的 lz4 版本仍然低于 1.8.3，可从源码编译安装更高版本的 lz4。

   ```bash
   # 需要 root 权限下载 lz4 源码（以 1.9.4 为例）
   wget https://github.com/lz4/lz4/releases/download/v1.9.4/lz4-1.9.4.tar.gz
   tar -zxf lz4-1.9.4.tar.gz
   cd lz4-1.9.4
   make
   make install
   # 安装完成后，验证 lz4 版本是否满足要求
   lz4 --version
   ```
