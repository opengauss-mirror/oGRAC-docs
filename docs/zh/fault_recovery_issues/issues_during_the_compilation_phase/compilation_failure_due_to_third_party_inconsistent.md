# 因三方库版本与 OS 版本不匹配导致编译失败的问题

## 一、问题现象

在 openEuler 20.03 或 24.03 系统上使用 22.03 版本的三方库，在执行 `sh local_install.sh compile -b debug` 或 `sh build_ograc.sh debug --with-dss` 编译 oGRAC 时，会出现如下类似报错信息:

```shell
# sh build_ograc.sh release --with-dss

BUILD_TYPE:Release
build dss start.
[INFO ] Third-party binarylibs search path: oGRAC
[INFO ] Current architecture: aarch64, binarylibs arch suffix: arm
[INFO ] openEuler 20.03 detected, use exact third-party binarylibs match.
Error: no valid third-party binarylibs found under: oGRAC
Expected directory: oGRAC/openGauss-third_party_binarylibs_openEuler_arm
Expected package:   oGRAC/openGauss-third_party_binarylibs_openEuler_arm.tar.gz
Expected directory: oGRAC/openGauss-third_party_binarylibs_openEuler_2003_arm
Expected package:   oGRAC/openGauss-third_party_binarylibs_openEuler_2003_arm.tar.gz
Please download and extract openGauss-third_party_binarylibs_* or specify the search path with --third-party-path <path>.
```

## 二、定位方法

查看编译回显，已经很明确没有找到正确的三方库二进制:

`Error: no valid third-party binarylibs found under: oGRAC`。

## 三、问题根因

oGRAC 增加编译前环境检查（相关逻辑位于 `build/build_dss.sh`），编译使用的三方库版本必须与 OS 版本严格对应。脚本通过读取 `/etc/os-release` 中的 `VERSION_ID` 获取系统版本，并据此查找匹配的三方库目录或压缩包。

| OS 版本 | 期望的三方库目录/包名 |
|---------|----------------------|
| openEuler 20.03 | `openGauss-third_party_binarylibs_openEuler_${arch}`<br>`openGauss-third_party_binarylibs_openEuler_2003_${arch}` |
| openEuler 22.03 | `openGauss-third_party_binarylibs_openEuler_2203_${arch}` |
| openEuler 24.03 | `openGauss-third_party_binarylibs_openEuler_2403_${arch}` |

其中 `${arch}` 为 `x86_64` 或 `arm` 等架构后缀。

如果在 openEuler 20.03 或 24.03 上使用了 22.03 版本的三方库（例如目录名为 `openGauss-third_party_binarylibs_openEuler_2203_x86_64`），脚本会判定该三方库与当前 OS 版本不匹配，进而报错并停止编译。

## 四、解决方法

1. 根据当前 OS 版本下载对应版本的三方库包。

   ```bash
   # 示例：openEuler 22.03 arm
   wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz

   # 示例：openEuler 24.03 arm
   wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2403_arm.tar.gz

   # 示例：openEuler 20.03 arm
   wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2003_arm.tar.gz
   ```

2. 将下载的三方库包解压到 oGRAC 代码目录内。

   ```bash
   cd /path/xxx/oGRAC
   tar -zxf openGauss-third_party_binarylibs_openEuler_xxx_xxx.tar.gz
   ```

3. 重新执行编译。

   ```bash
   cd /path/xxx/oGRAC/build
   sh build_ograc.sh debug/release --with-dss
   ```

   如果三方库不在默认路径下，可通过 `--third-party-path` 参数指定其所在目录。

   ```bash
   sh build_ograc.sh debug --with-dss --third-party-path /path/to/third/party/parent
   ```

4. 如需仅检查环境而不执行编译，可执行以下命令。

   ```bash
   sh build_ograc.sh debug --check-only
   # 或指定三方库路径
   sh build_ograc.sh debug --third-party-path /path/to/third/party/parent --check-only 
   ```
