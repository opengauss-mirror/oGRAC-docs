# Version Compilation

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:46:34.697Z pushedAt=2026-08-17T00:46:29.634Z -->

After the compilation environment is set up, you can follow the steps below to compile the version. Both debug and release versions are supported.

## Source Code Obtaining

```shell
chmod 755 -R compile_path
cd compile_path
git clone https://gitcode.com/opengauss/oGRAC.git
```

## Configuration Modification

If you need to disable the virtual memory protection option (recommended when compiling and installing the debug version):

```shell
cd oGRAC/build
sed -i 's/DUSE_PROTECT_VM=ON/DUSE_PROTECT_VM=OFF/g' Makefile.sh
```

## Compilation

### Single-Node Compilation

Currently, oGRAC single-node mode only supports standalone development and debugging, and does not support the packaging function.

```shell
cd build
sh local_install.sh prepare
sh local_install.sh compile -b debug
```

- `-b, --build_type=<type>`: Specifies the compilation type (`release`/`debug`, defaults to `release`)

#### Generated Directory

The output package is located at `oGRAC/oGRAC-DATABASE-*-64bit`.

### Two-Node Compilation and Packaging

#### Third-Party Dependency Download and Decompression

- Arm environment:

```bash
cd oGRAC/
wget --no-check-certificate https://opengauss.obs.cn-south-1.myhuaweicloud.com/6.0.0/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz
tar -zxf openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz
```

- CentOS environment:

```bash
cd oGRAC/
wget --no-check-certificate https://opengauss.obs.cn-south-1.myhuaweicloud.com/6.0.0/binarylibs/gcc10.3/openGauss-third_party_binarylibs_Centos7.6_x86_64.tar.gz
tar -zxf openGauss-third_party_binarylibs_Centos7.6_x86_64.tar.gz
```

#### Compilation Script Execution

```bash
cd oGRAC/build
# Basic usage (third-party libraries extracted in the oGRAC directory)
sh build_ograc.sh [release|debug] --with-dss

# Specify the directory containing third-party libraries (the path must be within the oGRAC directory, pointing to the parent directory that contains the third-party library folder)
sh build_ograc.sh [release|debug] --with-dss --third-party-path <path>
```

Parameter description:

| Parameter | Description |
|------|------|
| `release` / `debug` | Specify the compilation type (`release`: release version; `debug`: debug version). |
| `--with-dss` | Enable compilation of DSS-related components (mandatory for two-node or multi-node deployment). |
| `--third-party-path <path>` | Specify the search path for third-party dependencies, pointing to the parent directory that contains the `openGauss-third_party_binarylibs_*` folder. **The path must be within the oGRAC directory.** If not specified, the default search path is the root directory of oGRAC. |
| `-h`, `--help` | Display help information. |

> **Pre-compilation Environment Check**
>
> The build script has a built-in environment pre-check feature that automatically checks third-party library dependencies, OS version compatibility, and other items before compilation. If the third-party libraries have been extracted into the oGRAC directory, the script will locate them automatically without additional specification.
>
> If you need to verify the environment configuration in advance, you can also run the environment check separately (without performing compilation):
> 
> ```bash
> cd oGRAC/build
> sh build_dss.sh [release|debug] --check-only
> ```
>
> If the third-party libraries are not in the default path, you can specify the path (optional):
> 
> ```bash
> sh build_dss.sh [release|debug] <path> --check-only
> ```
>
> `--check-only` only performs the pre-compilation environment check without executing the actual compilation process.

#### Generated Directory

The output package is located in the `oGRAC/package/` directory.
