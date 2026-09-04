# Multi-Write Feature Overview

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:46:20.342Z pushedAt=2026-08-17T00:46:29.630Z -->

## 1. Multi-Write Capability

The oGRAC engine adopts a storage-compute separation architecture. It delivers transparent multi-read multi-write capabilities through distributed caching, transactional multi-version concurrency control(MVCC) mechanisms, and multi-primary cluster high availability technologies. Distributed Transparent Cluster (DTC) is the core architecture that underpins these capabilities. It consists of multiple key components working in coordination to ensure data consistency and transaction isolation during concurrent reads and writes across multiple nodes.

Key features of the transparent multi-write capability include:

- All nodes support reads and writes, with no distinction between primary and standby.
- Cross-node transaction consistency guarantee
- Distributed MVCC implementation
- Global SCN synchronization
- Distributed lock management
- High availability and automatic fault recovery

The oGRAC engine implements clustering functionality through distributed components (DRC, DCS, and DLS).

## 2. DMS Components

Distributed Management System (DMS) is the core of the transparent multi-write architecture and primarily includes the following features.

### 2.1 Cross-Node MVCC Implementation

The MVCC mechanism requires special handling in a distributed environment to ensure cross-node data consistency.

**Features**

- Supports cross-instance CR page construction
- Ensures version consistency through the global SCN mechanism

**Implementation**

- Each transaction has a unique global transaction ID.
- Data modifications generate new versions, which are recorded in the undo log.
- During cross-node queries, a visible data view is constructed through CR pages.

### 2.2 Cross-Node SCN Advancement

A system change number (SCN) is the key to ensuring global data consistency. The cross-node SCN advancement mechanism ensures that the SCN of all nodes within the cluster is monotonically increasing and eventually consistent.

**Implementation**

- Node SCN synchronization is achieved through a broadcast mechanism.
- A spinlock is used to control the atomicity of SCN updates.
- When the local SCN is smaller than the SCN broadcast by other nodes, the local SCN is automatically updated.
- Global consistency of the transaction commit order is ensured.

### 2.3 Distributed Lock Service (DLS)

DLS is responsible for managing lock resources across nodes, ensuring data consistency for concurrent operations.

**Implementation**

- Transmits lock requests across nodes through the MES messaging mechanism.
- Supports lock operation primitives such as spinlocks.
- Implements distributed lock wait and timeout mechanisms.
- Provides lock downgrade and deadlock detection capabilities.

### 2.4 Distributed Cache Service (DCS)

DCS is a key component in the transparent multi-write architecture responsible for managing cross-node page cache, page ownership, and CR page construction.

#### 2.4.1 DCS Core Concepts

**Page Ownership**

- Each data page has exactly one owner at any given moment.
- The owner node holds an exclusive lock on the page and is permitted to perform modification operations.
- Other nodes may hold a shared lock and are only permitted to perform read operations.

**Earlier Dirty Page (EDP) Mechanism**

- An EDP indicates that a node has a dirty page whose version is earlier than the current page version.
- It is used to track and manage page version dependencies in a distributed environment.
- It ensures the correctness of page flushing and checkpoint operations.

**Consistent Read (CR) Page Construction**

- CR pages are historical version pages constructed to meet the consistent read requirements of MVCC.
- CR page construction is supported for both heap tables (HEAP) and indexes (BTREE).
- CR pages are requested and transferred between nodes through a distributed messaging mechanism.

#### 2.4.2 DCS Message Structure

#### 2.4.3 DCS Core Features

**1. Page Request and Ownership Management**

DCS implements a sophisticated page request and ownership transfer mechanism:

- **Page request process**: The requester initiates a page request by first querying the primary node (Master) for the current owner of the page, and then requesting the page data from the owner.
- **Ownership transfer**: Upon receiving the request, the page owner transfers the page data to the requester and updates the lock status.
- **Batch operations**: Supports batch page requests and ownership claims to improve system throughput.

**2. Dirty Page Management and EDP Cleanup**

DCS provides a comprehensive dirty page management mechanism:

- **Dirty page tracking**: Tracks the dirty page status on nodes through the EDP bitmap.
- **Dirty page cleanup**: Cleans up global dirty pages during checkpoint.
- **Primary node coordination**: The primary node is responsible for coordinating cross-node dirty page cleanup.

**3. CR Page Construction and Consistent Read**

To support distributed MVCC, DCS implements cross-node CR page construction:

- **CR request processing**: Capable of processing different types of CR requests.
- **Primary node query**: Queries the page primary node to determine the CR construction location.
- **CR page construction**: The CR page is constructed on the node that holds the original page and then transferred to the requesting node.

#### 2.4.4 DCS Communication Mechanism

DCS employs a reliable message-based communication mechanism to ensure the correctness of data transmission between nodes:

**Message Retry Mechanism**

- Achieves reliable message delivery through the MES messaging mechanism.
- Supports configurable retry interval and retry count.
- Detects the status of target nodes to avoid sending messages to unavailable nodes.

**Timeout Control**

- Sets a message wait timeout to prevent prolonged blocking.
- Configures a separate timeout control for CR requests.

**Status Synchronization**

- Automatically update the cluster view when the node status changes.
- Detects the health status of nodes through the heartbeat mechanism.

#### 2.4.5 DCS Performance Optimization

Multiple performance optimization strategies are implemented in DCS:

- **Batch operations**: Batch page requests, batch ownership declarations, and batch release operations are supported.
- **Local priority**: Local cache is used preferentially to reduce network communication.
- **Concurrent processing**: Multi-threaded concurrent processing of DCS messages is supported.
- **Asynchronous operations**: Some operations support asynchronous execution, improving response speed.

#### 2.4.6 DCS Exception Handling

DCS is equipped with a comprehensive exception handling mechanism:

- **Resource cleanup**: Automatically cleans up related resources and locks upon node failure
- **Deadlock detection**: Provides a distributed deadlock detection and prevention mechanism
- **Logging**: Records exception details to facilitate problem diagnosis

As a core component of the transparent multi-write architecture, DCS ensures data consistency and efficient access in multi-node environments through sophisticated and refined distributed cache management. It thereby provides a solid foundation for the transparent multi-read multi-write capabilities of the oGRAC engine.

### 2.5 Distributed Resource Catalog (DRC)

DRC is a core component in the transparent multi-write architecture responsible for managing cluster-wide resource allocation, resource ownership, and resource migration. DRC ensures efficient utilization and consistent management of resources in a multi-node environment.

#### 2.5.1 DRC Core Concepts

- **Resource partition management**: DRC divides system resources into multiple partitions, each managed by a specific node, thereby enabling distributed resource management.
- **Resource ownership**: Each resource (such as a page or a lock) has an explicit ownership identifier that controls access permissions to the resource.
- **EDP mechanism**: Uses the EDP to track and manage dirty page status, ensuring data consistency.
- **Resource migration**: Enables smooth migration of resource ownership during the cluster Reform process.
- **Resource reclamation**: Automatically reclaims failed or timed-out resources, improving system stability.

#### 2.5.2 Core Data Structures

#### 2.5.3 Core Feature Implementation

1. **Resource ownership management**

2. **Resource partition management**

3. **DRC resource concentration mode**

DRC supports the resource concentration mode (`DRC_IN_REFORMER_MODE`). When all services run exclusively on the reformer node, this mode can be enabled to improve performance:

- All resources are centralized on the reformer node, reducing cross-node resource requests.
- This is controlled by modifying `DRC_IN_REFORMER_MODE`.
- It is suitable for scenarios where read and write loads are concentrated on specific nodes.

#### 2.5.4 Collaboration with Reform

DRC works closely with the cluster Reform mechanism to enable smooth resource migration when a node joins or leaves the cluster:

### 2.6 Cluster Reform Mechanism

Cluster reform is a key mechanism in the transparent multi-write architecture for ensuring high availability. It handles scenarios such as node joining, leaving, and failure, ensuring the consistency of cluster configuration and the continuity of services.

#### 2.6.1 Reform Core Concepts

- **Reformer node**: primary node responsible for coordinating the Reform process, typically the current primary node of the cluster.
- **Reform role**: The role played by a node during the Reform process (`STAY`/`LEAVE`/`JOIN`/`ABORT`)
- **Reform mode**: triggering method of Reform (planned/unplanned).
- **Resource reallocation**: redistribution of system resource ownership during the Reform process.
- **Partial recovery**: Only necessary data and states are recovered when a node joins or recovers.
- **Cluster view**: data structure that records the current cluster node status, version, and stability information.

#### 2.6.2 Reform States and Process

The main process of Reform includes:

1. **Preparation phase**: Determine the nodes participating in Reform and their respective roles.
2. **Communication channel establishment**: Ensure normal communication between nodes.
3. **Resource reallocation (Remaster)**: Reassign resource ownership.
4. **Data recovery**: Ensure data consistency across all nodes.
5. **Temporary channel closing**: Clean up temporary resources used during the Reform process.
6. **Service opening**: Allow client connections and requests.

#### 2.6.3 Detailed Implementation Process of Reform

##### 2.6.3.1 Reform Triggering Mechanism

Reform triggering is implemented through a dedicated triggering thread, and the main process is as follows:

When a cluster state change is detected, a function is used to determine whether Reform needs to be triggered:

##### 2.6.3.2 Reform State Machine

Reform implements a complex state machine to manage state transitions during the Reform process. The state machine is defined as follows:

The state machine determines the next operation based on a combination of states across four dimensions:

- Previous status of the node in CMS (`ONLINE`/`OFFLINE`)
- Subsequent status of the node in CMS
- Previous working state of the node (`JOINING`/`JOINED`/`LEAVING`/`LEFT`)
- Target working state of the node

Based on these state combinations, the state machine determines:

- List of operations to be executed (`REFORM_LIST_BEFORE`/`REFORM_LIST_AFTER`/`REFORM_LIST_JOIN`/`REFORM_LIST_LEAVE`/`REFORM_LIST_ABORT`)
- Whether it is a membership change operation
- Whether it is an impossible state transition (used for error detection)

##### 2.6.3.3 Primary Node Reform Process

The specific responsibilities of the primary node include:

- Establishing and maintaining the Reform communication channel
- Coordinating the resource reallocation process
- Collecting and synchronizing the status of all nodes
- Handling exception scenarios and error recovery
- Notifying all nodes that the reform is complete

##### 2.6.3.4 Reform Process on the Secondary Node

The primary responsibility of the secondary node during the reform process is to respond to instructions from the primary node and execute the corresponding operations:

##### 2.6.3.5 Cluster View Management

The cluster view is a critical data structure in the Reform process, used to track the cluster status:

The update and synchronization of the cluster view are handled by dedicated functions:

##### 2.6.3.6 Reform Completion Check

After the reform is completed, a series of checks and status updates are required:

#### 2.6.4 Reform Modes and Types

Reform supports multiple modes and types to accommodate different scenarios:

1. **Reform modes**:
   - `REFORM_MODE_PLANED`: Planned reform, such as normal node addition or removal
   - `REFORM_MODE_OUT_OF_PLAN`: Unplanned reform, such as node failure recovery
   - `REFORM_MODE_NONE`: No Reform state

2. **Node roles**:
   - `REFORM_ROLE_STAY`: Nodes that remain in the cluster
   - `REFORM_ROLE_JOIN`: Nodes that newly join the cluster
   - `REFORM_ROLE_LEAVE`: Nodes that leave the cluster
   - `REFORM_ROLE_ABORT`: Nodes that are abnormally terminated

3. **CMS status**:
   - `RC_CMS_ONLINE`: Online node
   - `RC_CMS_OFFLINE`: Offline node

4. **Working status**:
   - `RC_JOINING`: The node is joining the cluster.
   - `RC_JOINED`: The node has joined the cluster.
   - `RC_LEAVING`: The node is leaving the cluster.
   - `RC_LEFT`: The node has left the cluster.

#### 2.6.5 Exception Handling and Fault Tolerance

The Reform mechanism is designed with comprehensive exception handling and fault tolerance mechanisms:

1. **State repair**: Repairs possible state inconsistency issues.

2. **Timeout handling**: Avoids deadlocks using lock operations with timeouts.

3. **Failure retry**: Implements automatic retry through macros for critical operations.

4. **Bitmap synchronization**: Ensures the consistency of the node status bitmap.

5. **Error detection and recovery**: Performs exception handling through error codes and status checks.

#### 2.6.6 Collaboration with DRC

Reform works closely with DRC to achieve smooth resource migration and redistribution.

During the Reform process, the collaboration flow between Reform and DRC is as follows:

1. Reform triggers the resource redistribution (Remaster) process.
2. DRC coordinates each node to prepare for redistribution.
3. The primary node computes a new resource allocation plan.
4. Each node performs resource migration.
5. After the migration is complete, confirmation is sent to notify Reform.
6. Reform proceeds with the subsequent Reform steps.

#### 2.6.7 Detailed Implementation Process of Remaster

The Remaster process is a core component of the Reform mechanism, responsible for reallocating and migrating resources when node roles or cluster topology changes. The following describes the implementation mechanism of Remaster in detail.

##### 2.6.7.1 Core Management Structure of Remaster

##### 2.6.7.2 Four Phases of the Remaster Process

1. **Preparation phase (REMASTER_PREPARE)**
   - Initialize the Remaster manager.
   - Clean up resources of the departed node.
   - Wait for all nodes to be ready.

2. **Task assignment phase (REMASTER_ASSIGN_TASK)**
   - The Master node assigns partition tasks based on the new cluster topology.
   - The export node and import node for each partition are determined.

3. **Resource migration phase (REMASTER_MIGRATE)**
   - Perform buffer resource migration.
   - Execute lock resource migration.
   - Wait for migration to complete.

4. **Cleanup and completion phase**
   - Release resources after migration.
   - Confirm that all tasks are completed.
   - Update the cluster status.

##### 2.6.7.3 Core Implementation Functions

#### 2.6.8 Detailed Implementation of the Log Replay Process

The log replay process is responsible for restoring data consistency during the Reform process, especially in the event of node failures or topology changes. The implementation mechanism of log replay is described in detail below:

##### 2.6.8.1 Core Log Replay Process

1. **Log read phase**
   - Read redo logs from multiple nodes in parallel.
   - Load log batches into memory buffers.

2. **Log analysis phase (ANALYSIS)**
   - Analyze log batches and determine the pages that need to be recovered.
   - Build the recovery dataset.

3. **Log replay phase (RECOVERY)**
   - Replay log groups in parallel.
   - Apply changes to data pages.

4. **Consistency verification phase**
   - Verify data consistency.
   - Update the SCN and recovery point.

#### 2.6.9 Reform Performance Monitoring

oGRAC provides a comprehensive set of Reform performance monitoring metrics, facilitating the tracking and analysis of the Reform process:

- Duration of the preparation phase
- Duration of the recovery phase
- Duration of resource migration

## 3. Introduction to DSS

### 3.1 DSS Core Features

Distributed Storage System (DSS) provides underlying storage management capabilities, delivering a stable data storage foundation for the transparent multi-write architecture.

**Features**

- Unified file operation APIs
- Distributed storage access
- Multi-node data sharing

**Implementation**

- Storage access is provided through the dynamic library `libdssapi.so`.
- Complete file operations are supported: creation, read/write, deletion, renaming, and more.
- Asynchronous I/O (AIO) operations are implemented to enhance performance.
- Provides log callback and connection management mechanisms

**Main APIs**

```c
// Core file operation APIs
dss_fcreate — Creates a file.
dss_fopen — Opens a file.
dss_fread — Reads data from a file.
dss_fwrite — Writes data to a file.
dss_fclose — Closes a file.
dss_fremove — Removes a file.
dss_frename — Renames a file.

// Asynchronous I/O APIs
dss_aio_prep_pread — Prepares an asynchronous pre-read operation.
dss_aio_prep_pwrite — Prepares an asynchronous pre-write operation.
dss_aio_post_pwrite — Posts an asynchronous write operation.
```

**Log Levels**

- `DSS_LOG_LEVEL_ERROR`: error level
- `DSS_LOG_LEVEL_WARN`: warning level
- `DSS_LOG_LEVEL_INFO`: information level
