# Product Description

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:45:52.899Z pushedAt=2026-08-17T00:46:29.618Z -->

## Overview

openGauss Real Application Cluster (oGRAC) is the multi-primary database architecture of openGauss. It manages all database instances in the cluster and supports globally real-time consistent read and write operations, thereby improving the cluster's read/write throughput, scalability, and high availability. The core concept is as follows: the database provides a kernel that can be easily extended into a multi-primary architecture; database instances achieve cross-node transaction and page cache consistency through the Distributed Memory Service (DMS); and all database instances share a single copy of data storage, enabling simultaneous read and write operations across all nodes with shared storage.

## Positioning

Based on an innovative database kernel, oGRAC delivers real-time high-concurrency transaction processing, sub-second Recovery Time Objective (RTO), and a performance scaling factor greater than 0.8, making it suitable for powering core systems in industries such as finance, telecommunications, and public sectors.

## Features

### Multi-Read Multi-Write

The primary feature of oGRAC is its support for multi-read multi-write, which is achieved through the following components:

- Distributed Resource Catalog (DRC): Manages distributed metadata, including the owning node information, permissions, and concurrency control of resources such as data pages and global locks across the cluster.

- Distributed Cache Service (DCS): Manages the concurrency of data pages across the cluster, including concurrency control for page reads and writes, processing of intra-cluster page read and write requests, management of the latest version holders and read-only copy holders of data pages, cross-node cached page transfer, and CR page construction.

- Distributed Lock Service (DLS): Manages the concurrency of lock resources across the cluster, including the concurrent processing of lock resources such as spinlocks, latches, and objects among nodes.

- Distributed Storage Service (DSS): Provides underlying storage management capabilities, offering a unified interface for the database kernel to access the underlying storage; and manages different forms of underlying storage, including distributed and centralized storage.

- Distributed MVCC: Each transaction is assigned a unique global transaction ID. When data is modified, a new version is generated directly, while the old version and transaction information are recorded in the undo log. During cross-node queries, a visible data view is constructed through CR page building. Upon transaction commit, node SCN synchronization is achieved via a broadcast mechanism. When the local SCN is smaller than the SCN broadcast by other nodes, the local SCN is automatically updated, ensuring global consistency of the transaction commit order.

- Failure recovery (Remaster/Recovery): When a node fails, the redistribution and reconstruction of global DRC and other information are handled. Meanwhile, the redo and undo of the failed node are processed — redo logs after the checkpoint are replayed, and uncommitted transactions are rolled back — to restore the cluster to a consistent state.

All nodes in the oGRAC cluster share the cluster control file (Ctrl), system tablespace, doublewrite area, and user tablespaces. Each node has its own independent undo, temp tablespace, and redo. Each node can only write to its own undo, redo, and temp files. During normal cluster operation, a node may read the undo files of other nodes to construct CR pages, but cannot write to the undo files of other nodes. During cluster failure recovery, the node performing the recovery reads the redo logs of the failed node. During normal cluster operation, each node can only write to its own redo. The temp partition is also node-private and can only be read by the local node.

### High-Performance SQL Engine

- Supports rule-based logical optimization and cost-based optimizer (CBO).

- Generates execution plans with minimal physical cost based on dynamic programming.

- Supports multiple join operators such as NEST LOOP, HASH, and MERGE.

- Supports execution plan caching and dynamic sampling, with the execution engine supporting parallel query.

### Cluster Management

Cluster Management Service (CMS) is mainly responsible for cluster monitoring and management. Its functions include maintaining cluster member status information, monitoring whether the status of all resources in the cluster is normal, handling abnormal faults, performing cluster redistribution, and conducting arbitration, thereby ensuring the consistency of cluster configurations and the continuity of services.
