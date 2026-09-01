# JDBC Multi-Node Load Balancing

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-20T08:47:02.905Z pushedAt=2026-07-20T09:02:37.310Z -->

> [!NOTE]
> In the following example scenarios, `node` represents `host:port`, where `host` is the database server name or IP address, and `port` is the database server port.

## JDBC Load Balancing Feature Description

JDBC can access a cluster by configuring multiple database nodes in the URL. By setting the load balancing parameter `autoBalance` in the URL, JDBC follows specific load balancing rules when creating connections. It then establishes client connections to the cluster sequentially on each node configured in the URL, thereby achieving connection load balancing.
The default value is `false`, which means load balancing mode is disabled, and JDBC always establishes a connection with the first node in the URL that can be successfully connected to. Currently, JDBC provides four load balancing modes: `roundrobin`, `priority roundrobin`, `leastconn`, and `shuffle`. Examples are as follows:

- `roundrobin`: This mode establishes connections with the candidate nodes in the URL in turn. The valid values are `roundrobin`, `true`, and `balance`. If a client wants to use this mode to connect to a three-node cluster and rotate through the nodes when creating multiple connections, the following configuration can be used:

  ```plaintext
  jdbc:oGRAC://node1,node2,node3?autoBalance=roundrobin
  ```

- `shuffle`: This mode randomly selects a node from the URL to establish a connection each time. The valid value is `shuffle`. A reference configuration for connecting to a cluster in this mode is as follows:

  ```plaintext
  jdbc:oGRAC://node1,node2,node3?autoBalance=shuffle
  ```

- `leastconn`: This mode prioritizes candidate nodes based on the number of active connections on each node, preferring nodes with fewer connections. It tracks connections established through the current driver within the current cluster using the `leastconn` mode. The valid value is `leastconn`. A reference configuration for connecting to a cluster in this mode is as follows:

  ```plaintext
  jdbc:oGRAC://node1,node2,node3?autoBalance=leastconn
  ```

- `priority roundrobin`: This mode preferentially performs round-robin connection establishment on the first n candidate nodes. The valid value is `priority[n]`, where n is a non-negative integer. Taking a three-node cluster as an example, if the client wants to preferentially run services on node 1 and node 2, with node 3 serving only as a standby node when other nodes are abnormal, this parameter can be set as follows:

  ```plaintext
  jdbc:oGRAC://node1,node2,node3?autoBalance=priority2
  ```

> [!NOTE]
>
> - The load balancing supported by JDBC is driver-level load balancing. It balances the load of connections created on the same cluster based on this driver, and does not work based on the actual number of connections on each node in the cluster, nor does it balance connections created by other drivers.
