
### Functional Requirements
- put(key, value) -> writes data to the cache with the given key
- get(key) -> retrieves data from the cache with the given key

### Non Functional Requirements
- Highly scalable - should be able to support large no. of data
- Highly Available - system should be able to respond even during n/w failure
- Fault tolerant
- Eventual consistency
- system should be able to handle read and write requests with low latency

### High level design
- start with the local cache (in-memory cache) in a single application server
- client sends a write request to the AS and the data is stored in in-memory cache
- client sends a read request to the AS and the data is retrieved from the in-memory cache
- in-memory cache can be implemented using hash map
- Cons -
	- local cache has limited capacity so you cannot store large amount of data and it is difficult to scale
	- solution - distributed cache (distributes the data across multiple nodes)

### Where to host the cache ?
- Dedicated Cache Cluster
	- separate servers or nodes are used for caching
	- AS and CS are separate
	- Both app and cache servers can be scaled independently
	- they don't share the resources
	- multiple app servers can use the cache servers
- Co-located cache
	- app servers and cache servers are located in the same server or node
	- they can scale together by adding more servers but cannot scale independently
	- share the same resources

### Deep Dive
- we need multiple cache servers to handle large amount of data (choose dedicated cache cluster)
- data needs to be distributed across multiple cache servers how ???? (using consistent hashing)
- #### How data will be stored and retrieved in cache servers ?
	- hash both the cache nodes and the keys using the same hash function to find the position on the virtual ring
	- traverse the ring in clockwise direction starting from the position of the key until a node is found, the data object is stored or retrieved in or from that node
- #### How the requests get routed to the dedicated cache server ?
	- cache client routes the request to the dedicated cache node to store or retrieve the data object 
	- cache client knows about all the cache nodes
	- all cache client should have the same list of cache nodes 
	- How does a cache client gets this list of cache nodes ???? 
		- from the Configuration Service
		- cache nodes registers itself with the configuration service or the service registry
		- cache client gets this information from the configuration service
		- when cache nodes do not send heartbeat signal to the configuration service for extended period of time, it deregisters the cache node from the configuration service and cache client will not have this node in its list
		- cache client knows which request needs to be routed to which cache node for a particular key
- #### How to achieve high availability and reliability ?
	- using Replication
	- data should be replicated asynchronously to N nodes in the system to guarantee high availability and eventual consistency
	-  use multi-leader or leaderless replication
	- once key is located on the hash ring start clockwise from the position of the ring and choose the first N servers or nodes on the hash ring
	- because of the virtual nodes on the hash ring, first N nodes might not be the first N distinct physical nodes (might be less than N), so always choose the distinct nodes
	- for high availability replicas are stored in different data centres
	- main problem with distributed cache is maintaining consistency
- #### How consistency is maintained across different replicas ?
	- consistency can be guaranteed using quorum technique
	- when a write request comes to a node it is replicated to the remaining N-1 nodes in the system but the system waits for only W-1 nodes before returning success
	- when a read request comes to a node it is read from the remaining N-1 nodes in the system but the system waits for only R-1 nodes before returning success
	- as the system is allowing concurrent write requests, replicas will have inconsistent data
	- replicas with stale data are updated with new data during read operation -> read repair
- #### How to resolve concurrent write requests ?
	- using versioning technique and vector clock
	- when concurrent write requests comes new versions are generated
	- when read requests comes multiple conflicting versions are returned to the application
	- application resolves the conflicting versions using merging technique like CRDTs and writes back the data
	- conflict resolution are done on the client side
- #### How to detect node failures ?
	- Gossip Protocol
- #### How node failures are handled ?
	- Temporary failures -> sloppy quorum and hinted handoff
	- Permanent failure -> Anti entropy using Merkle trees
