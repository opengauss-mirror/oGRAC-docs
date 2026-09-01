# Compilation Failure Caused by Third‑Party Library and OS Version Mismatch

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:07.355Z pushedAt=2026-08-17T00:46:29.647Z -->

## 1. Symptom

When a third-party library of version 22.03 is used on an openEuler 20.03 or 24.03 system, the following error message appears during the compilation of oGRAC by executing `sh local_install.sh compile -b debug` or `sh build_ograc.sh debug --with-dss`:

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

## 2. Issue Locating

By examining the compilation output, it is evident that the correct third-party binary library was not found:

`Error: no valid third-party binarylibs found under: oGRAC`.

## 3. Root Cause

oGRAC introduces a pre-compilation environment check (the relevant logic is located in `build/build_dss.sh`), which requires that the version of the third-party library used for compilation must strictly match the OS version. The script obtains the system version by reading `VERSION_ID` from `/etc/os-release`, and then searches for the matching third-party library directory or compressed package accordingly.

| OS Version | Expected Third-Party Library Directory/Package Name |
|---------|----------------------|
| openEuler 20.03 | `openGauss-third_party_binarylibs_openEuler_${arch}`<br>`openGauss-third_party_binarylibs_openEuler_2003_${arch}` |
| openEuler 22.03 | `openGauss-third_party_binarylibs_openEuler_2203_${arch}` |
| openEuler 24.03 | `openGauss-third_party_binarylibs_openEuler_2403_${arch}` |

Where `${arch}` is an architecture suffix such as `x86_64` or `arm`.

If a third-party library of version 22.03 is used on openEuler 20.03 or 24.03 (for example, the directory is named `openGauss-third_party_binarylibs_openEuler_2203_x86_64`), the script will determine that the third-party library does not match the current OS version, and will then report an error and halt the compilation.

## 4. Solution

1. Download the third-party library package corresponding to the current OS version.

   ```bash
   # Example: openEuler 22.03 arm
   wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz

   # Example: openEuler 24.03 arm
   wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2403_arm.tar.gz

   # Example: openEuler 20.03 arm
   wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/latest/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2003_arm.tar.gz
   ```

2. Extract the downloaded third-party library package into the oGRAC code directory.

   ```bash
   cd /path/xxx/oGRAC
   tar -zxf openGauss-third_party_binarylibs_openEuler_xxx_xxx.tar.gz
   ```

3. Re-execute the compilation.

   ```bash
   cd /path/xxx/oGRAC/build
   sh build_ograc.sh debug/release --with-dss
   ```

   If the third-party libraries are not located in the default path, you can specify their directory using the `--third-party-path` parameter.

   ```bash
   sh build_ograc.sh debug --with-dss --third-party-path /path/to/third/party/parent
   ```

4. To check the environment only without performing compilation, execute the following command.

   ```bash
   sh build_ograc.sh debug --check-only
   # Or specify the third-party library path
   sh build_ograc.sh debug --third-party-path /path/to/third/party/parent --check-only 
   ```
