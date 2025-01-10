# Zookeeper

| |Index|
|---|---|
|1|[Roles](#role)|
|2|[Data](#data)|
|3|[Update](#update)|
|4|[Session](#session)|
|5|[Guarantees](#guarantee)|
|6|[Commands](#command)|

ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them ,which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

ZooKeeper allows distributed processes to coordinate with each other through a shared hierarchal namespace which is organized similarly to a standard file system.The name space consists of data registers - called znodes, in ZooKeeper parlance - and these are similar to files and directories. Unlike a typical file system, which is designed for storage, ZooKeeper data is kept in-memory, which means ZooKeeper can acheive high throughput and low latency numbers.

## 1 <a id='role'></a>Roles
- The servers that make up the ZooKeeper service must all know about each other. They maintain an in-memory image of state, along with a transaction logs and snapshots in a persistent store. As long as a majority of the servers are available, the ZooKeeper service will be available.
- Clients connect to a single ZooKeeper server. The client maintains a TCP connection through which it sends requests, gets responses, gets watch events, and sends heart beats. If the TCP connection to the server breaks, the client will connect to a different server.

![Server](https://github.com/barneywill/hadoop_suite/blob/main/imgs/zk_server.jpg)

## 2 <a id='data'></a>Data
- The name space provided by ZooKeeper is much like that of a standard file system. A name is a sequence of path elements separated by a slash (/). Every node in ZooKeeper's name space is identified by a path.
- Unlike is standard file systems, each node in a ZooKeeper namespace can have data associated with it as well as children. It is like having a file-system that allows a file to also be a directory. 
- ZooKeeper also has the notion of ephemeral nodes. These znodes exists as long as the session that created the znode is active. When the session ends the znode is deleted. 
- ZooKeeper supports the concept of watches. Clients can set a watch on a znodes. A watch will be triggered and removed when the znode changes. When a watch is triggered the client receives a packet saying that the znode has changed. And if the connection between the client and one of the Zoo Keeper servers is broken, the client will receive a local notification.

![Data](https://github.com/barneywill/hadoop_suite/blob/main/imgs/zk_data.jpg)

## 3 <a id='update'></a>Update
- Every ZooKeeper server services clients. Clients connect to exactly one server to submit irequests. Read requests are serviced from the local replica of each server database. Requests that change the state of the service, write requests, are processed by an agreement protocol.
- As part of the agreement protocol all write requests from clients are forwarded to a single server, called the leader. The rest of the ZooKeeper servers, called followers, receive message proposals from the leader and agree upon message delivery. The messaging layer takes care of replacing leaders on failures and syncing followers with leaders.

![Update](https://github.com/barneywill/hadoop_suite/blob/main/imgs/zk_update.jpg)

## 4 <a id='session'></a>Session
- A ZooKeeper client establishes a session with the ZooKeeper service by creating a handle to the service using a language binding. Once created, the handle starts of in the CONNECTING state and the client library tries to connect to one of the servers that make up the ZooKeeper service at which point it switches to the CONNECTED state. During normal operation will be in one of these two states. If an unrecoverable error occurs, such as session expiration or authentication failure, or if the application explicitly closes the handle, the handle will move to the CLOSED state. 

![Session](https://github.com/barneywill/hadoop_suite/blob/main/imgs/zk_session.jpg)

## 5 <a id='guarantee'></a>Guarantees
- Sequential Consistency - Updates from a client will be applied in the order that they were sent.
- Atomicity - Updates either succeed or fail. No partial results.
- Single System Image - A client will see the same view of the service regardless of the server that it connects to.
- Reliability - Once an update has been applied, it will persist from that time forward until a client overwrites the update.
- Timeliness - The clients view of the system is guaranteed to be up-to-date within a certain time bound.

## 6 <a id='command'></a>Commands
ZooKeeper Commands: The Four Letter Words
```
$ echo mntr | nc localhost 2185

conf
# Print details about serving configuration.
cons
# List full connection/session details for all clients connected to this server. Includes information on numbers of packets received/sent, session id, operation latencies, last operation performed, etc...
crst
# Reset connection/session statistics for all connections.
dump
# Lists the outstanding sessions and ephemeral nodes. This only works on the leader.
envi
# Print details about serving environment
ruok
# Tests if server is running in a non-error state. The server will respond with imok if it is running. Otherwise it will not respond at all.
srst
# Reset server statistics.
srvr
# Lists full details for the server.
stat
# Lists brief details for the server and connected clients.
wchs
# Lists brief information on watches for the server.
wchc
# Lists detailed information on watches for the server, by session. This outputs a list of sessions(connections) with associated watches (paths). Note, depending on the number of watches this operation may be expensive (ie impact server performance), use it carefully.
wchp
# Lists detailed information on watches for the server, by path. This outputs a list of paths (znodes) with associated sessions. Note, depending on the number of watches this operation may be expensive (ie impact server performance), use it carefully.
mntr
# Outputs a list of variables that could be used for monitoring the health of the cluster.
```