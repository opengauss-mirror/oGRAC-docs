# oGRAC搭建存储设备挂载配置完整操作指南

## 一、服务器端 iSCSI 基础配置

### 步骤 1：查看本机 iSCSI 启动器 IQN

执行以下命令查看服务器本地 IQN 标识，该标识需后续在磁阵后台绑定：

```bash
cat /etc/iscsi/initiatorname.iscsi
```

**输出示例**：

```plain
InitiatorName=iqn.2012-01.com.openeuler:存储IP
```

### 步骤 2：安装 iSCSI 客户端并启动服务

若服务器未安装 iSCSI 客户端工具，执行以下命令安装并启动核心服务：

```bash
# 安装iSCSI客户端工具
yum install -y iscsi-initiator-utils

# 启动iscsid核心服务
systemctl start iscsid

# 启动iscsi登录管理服务
systemctl start iscsi
```

### 步骤 3：发现磁阵存储目标

通过磁阵IP扫描存储设备目标，替换命令中【磁阵IP】为实际存储地址：

```bash
iscsiadm -m discovery -t sendtargets -p 存储IP
```


### 步骤 4：登录磁阵存储节点

完成发现与认证后，登录磁阵节点建立连接：

```bash
iscsiadm -m node -T 目标名 -p 磁阵IP -l
```

## 二、后台配置


### 步骤 1：接入存储Device Manager

在网页后台输入磁阵 Device Manager 管理地址，进入存储管理界面。

### 步骤 2：添加主机启动器

1. 后台依次点击：**服务 → 主机组 → 启动器 → iSCSI（或FC）**；

2. 回到服务器端，再次执行 `cat /etc/iscsi/initiatorname.iscsi` 获取主机IQN；

3. 核对后台启动器列表中是否已同步服务器IQN，未存在则手动添加。

### 步骤 3：创建主机及LUN资源

1. 在后台创建主机，选择刚刚添加的iSCSI启动器进行绑定；

2. 依次点击：**服务 → LUN组 → 创建**，根据业务规格创建对应LUN；

3. LUN创建完成后，配置映射关系，绑定至已创建的主机/主机组。

## 三、服务器端验证LUN挂载结果

完成后台配置后，回到服务器刷新SCSI总线，识别新增存储设备：

```bash
# 刷新SCSI设备总线
rescan-scsi-bus.sh

# 查看iSCSI存储设备
lsscsi -is
```

可通过LUN对应的WWN信息，核对服务器是否成功识别、挂载新创建的磁阵LUN，确认配置生效。
