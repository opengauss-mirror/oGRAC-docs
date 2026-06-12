# 版本编译

在搭建好编译环境后，便可以安装下面的步骤进行版本编译，可以编译 debug 和 release 版本。

## 获取源码

```shell
chmod 755 -R compile_path
cd compile_path
git clone https://gitcode.com/opengauss/oGRAC.git
```

## 配置修改

如需关闭保护虚拟内存选项(如果编译安装的是debug版本建议关闭保护虚拟内存选项)：

```shell
cd oGRAC/build
sed -i 's/DUSE_PROTECT_VM=ON/DUSE_PROTECT_VM=OFF/g' Makefile.sh
```

## 编译

### 单节点编译

当前oGRAC单节点仅支持单机开发调试，不支持打包功能。

```shell
cd build
sh local_install.sh prepare
sh local_install.sh compile -b debug
```

- `-b, --build_type=<type>`：指定编译类型（release/debug，默认release）

#### 生成目录

输出包位于：`oGRAC/oGRAC-DATABASE-*-64bit`

### 两节点编译出包

#### 下载3方依赖并解压

- arm环境：

```bash
cd oGRAC/
wget --no-check-certificate https://opengauss.obs.cn-south-1.myhuaweicloud.com/6.0.0/binarylibs/gcc10.3/openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz
tar -zxf openGauss-third_party_binarylibs_openEuler_2203_arm.tar.gz
```

- Centos环境：

```bash
cd oGRAC/
wget --no-check-certificate https://opengauss.obs.cn-south-1.myhuaweicloud.com/6.0.0/binarylibs/gcc10.3/openGauss-third_party_binarylibs_Centos7.6_x86_64.tar.gz
tar -zxf openGauss-third_party_binarylibs_Centos7.6_x86_64.tar.gz
```

#### 执行编译脚本

```bash
cd oGRAC/build
# 基本用法（三方库解压在 oGRAC 目录内）
sh build_ograc.sh [release|debug] --with-dss

# 指定三方库所在目录（路径必须在 oGRAC 目录内，指向包含三方库文件夹的父目录即可）
sh build_ograc.sh [release|debug] --with-dss --third-party-path ${OGDB_CODE_PATH}
```

参数说明：

| 参数 | 说明 |
|------|------|
| `release` / `debug` | 指定编译类型（release：发布版本；debug：调试版本） |
| `--with-dss` | 启用 DSS 相关组件编译（两节点及以上部署必须） |
| `--third-party-path <path>` | 指定三方库依赖的查找路径，指向包含 `openGauss-third_party_binarylibs_*` 文件夹的父目录即可，**路径必须在 oGRAC 目录内**；不指定时默认在 oGRAC 目录根目录查找 |
| `-h`, `--help` | 显示帮助信息 |

> **编译前环境检查**
>
> 编译脚本已内置环境预检功能，会在编译前自动检查三方库依赖、OS 版本兼容性等。如三方库已解压到 oGRAC 目录内，脚本会自动查找，无需额外指定。
>
> 如需提前验证环境配置是否正确，也可单独运行环境检查（不执行编译）：
> ```bash
> cd oGRAC/build
> sh build_dss.sh [release|debug] --check-only
> ```
>
> 如果三方库不在默认路径，可通过 `--third-party-path` 指定（该参数非必填）：
> ```bash
> sh build_dss.sh [release|debug] --check-only --third-party-path <path>
> ```
>
> `--check-only` 仅执行编译前环境检查，不执行实际编译流程。

#### 生成目录

输出包位于：`oGRAC/package/` 目录下
