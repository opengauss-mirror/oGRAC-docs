# Common JDBC Parameters<a name="ZH-CN_TOPIC_0000001399380057"></a>

## autoBalance

**Function**: JDBC can access multiple database nodes by configuring them in the URL. By setting a load balancing algorithm, JDBC can establish client-node connections on each node configured in the URL in turn according to specific rules, thereby achieving load balancing across connections. When the parameter is set to `false`, JDBC always establishes a connection with the same node in the URL that meets the connection establishment conditions.

Currently, JDBC provides four load balancing modes: `roundrobin`, `priority roundrobin`, `leastconn`, and `shuffle`.

- `roundrobin`: This mode establishes connections with candidate nodes in turn. The valid values are `roundrobin`, `true`, and `balance`.

- `priority roundrobin`: This mode preferentially performs round-robin connection establishment on the first n candidate nodes. The valid value is `priority[n]`, where n is a non-negative integer.

- `leastconn`: This mode prioritizes candidate nodes based on the number of active connections on each node, preferring nodes with fewer connections. It tracks connections established through the current driver within the current cluster using the `leastconn` mode. The valid value is `leastconn`.

- `shuffle`: This mode randomly selects a candidate node to establish a connection. The valid value is `shuffle`.

**Recommendation**: The default value is `false`.

## fetchsize

**Principle**: When `fetchsize` is set to `n`, the following behavior applies after the database server executes a query. Each time the caller invokes `resultset.next()`, JDBC first communicates with the server to fetch `n` rows of data into the JDBC client. It then returns the first row to the caller. When the caller retrieves the (n+1)-th row, JDBC again fetches data from the database server.

**Function**: This prevents the database from transmitting all results to the client at once, which would otherwise consume excessive memory resources on the client.

**Recommendation**: Configure this parameter based on the volume of queried data and the memory capacity of the client machine. The default value is `0`, which indicates that all data is queried.

## loginTimeout

**Function**: It controls the duration for establishing a socket connection to the database. The connection attempt is terminated when the threshold is exceeded.

**Recommendation**: The default value is `0`, which indicates an indefinite wait for the server to respond.

## socketTimeout

**Function**: It controls the socket operation timeout value. If the execution of a service statement or the reading of a data stream from the network exceeds this threshold (that is, when an SQL statement has been executing beyond the specified time without returning data), the connection is terminated.

**Recommendation**: This parameter limits the maximum execution time of a single statement. If the execution of a single statement exceeds this value, it times out and an error is reported. Configure this parameter based on service characteristics.

## receiveBufferSize

**Function**: It specifies the data size of the socket read buffer.

**Recommendation**: The default value is `-1`, which indicates that the buffer size is not limited.

## sendBufferSize

**Function**: It specifies the data size of the socket write buffer.

**Recommendation**: The default value is `-1`, which indicates that the buffer size is not limited.
