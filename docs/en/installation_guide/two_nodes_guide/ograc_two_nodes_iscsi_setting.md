# Operation Guide for Storage Device Mounting Configuration in oGRAC Setup

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:49:06.694Z pushedAt=2026-08-17T00:46:29.690Z -->

## 1. Server-Side Basic iSCSI Configuration

### Step 1: Viewing the Local iSCSI Qualified Name (IQN)

Run the following command to view the local IQN of the server. This identifier must be bound on the storage array backend subsequently.

```bash
cat /etc/iscsi/initiatorname.iscsi
```

**Output example**:

```plain
InitiatorName=iqn.2012-01.com.openeuler:Storage IP address
```

### Step 2: Installing the iSCSI Client and Starting the Service

If the iSCSI client tool is not installed on the server, run the following commands to install and start the core service:

```bash
# Install the iSCSI client tool.
yum install -y iscsi-initiator-utils

# Start the iscsid service.
systemctl start iscsid

# Start the iSCSI login management service.
systemctl start iscsi
```

### Step 3: Discovering Storage Array Targets

Scan for storage device targets using the storage array IP address. Replace `Storage IP address` in the command with the actual storage IP address:

```bash
iscsiadm -m discovery -t sendtargets -p Storage IP address
```

### Step 4: Logging In to the Storage Array Node

After discovery and authentication are complete, log in to the storage array node to establish a connection:

```bash
iscsiadm -m node -T Target name -p Storage array IP address -l
```

## 2. Backend Configuration

### Step 1: Accessing Device Manager

Open a web browser and enter the Device Manager management address of the storage array to access the storage management interface.

### Step 2: Adding the Host Initiator

1. In the management console, choose **Services > Host Groups > Initiators > iSCSI (or FC)**.

2. Return to the server and run `cat /etc/iscsi/initiatorname.iscsi` again to obtain the host IQN.

3. Check whether the server IQN has been synchronized in the initiator list of the management console. If not present, add it manually.

### Step 3: Creating a Host and LUN Resources

1. Create a host in the management console and bind it with the iSCSI initiator you just added.

2. Choose **Services > LUN Groups > Create**, and create the corresponding LUNs based on the service specifications.

3. After the LUNs are created, configure the mapping relationship and bind them to the created host or host group.

## 3. LUN Mount Verification on the Server

After completing the management console configuration, return to the server and rescan the SCSI bus to identify the newly added storage devices:

```bash
# Rescan the SCSI device bus.
rescan-scsi-bus.sh

# View iSCSI storage devices.
lsscsi -is
```

Use the WWN information associated with the LUNs to verify that the server has successfully discovered and mounted the newly created LUNs, thereby confirming that the configuration has taken effect.
