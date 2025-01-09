### Why to use distributed cache ?
- Scaling
- Reduce latency
- Reduce load on database
- speed up expensive queries
- Fault Tolerant

### Disadvantages of Single Cache
- not scalable - memory is limited, as data volume grows it becomes difficult to scale the cache
- single point of failure - as there is no replication
- reduced performance - unable to handle spike in traffic

### Distributed Cache
- data is stored across multiple cache servers
- Benefits -
- horizontally scalable - by adding more cache servers as data volume increases
- fault tolerant - as data is replicated across multiple cache servers so a single node failure doesn't affect the entire cache and the system continues to serve requests (improves availability)
- Performance optimization - by storing the data across multiple nodes, it improves the response time and reduces latency

### Use Cases
- High traffic during peak events in Ecommerce site - eg - Black Friday sale
- Live sports streaming

### Different hosting options
- different options to host distributed caches
- Dedicated cache servers
	- cache is hosted in standalone servers separate from application servers
	- Advantages
		- Scalability - as each cache server is a standalone server so it can be scaled independently based on demands
		- Resource Isolation - prevents resource consumption from application servers
	- Disadvantages
		- Network Latency - as n/w call is there b/w application and the cache servers therefore latency is high
		- Higher costs - operating cache servers can be expensive when handling high traffic surges
- Colocated caching
	- cache and application service are located in the same server
	- Advantages
		- Low Latency - as there is no n/w call 
		- Cost efficiency - as no need to maintain separate cache servers
	- Disadvantages
		- Low scalability - difficult to scale cache as you need to scale the entire server that hosts both the cache and the application
		- Resource contention - the same resource (CPU, memory, I/O) is shared by both the cache and the app server
- Cloud based caching services
	- e.g - Amazon Elasticache is a fully managed cloud based caching solution
	- supports Redis and Memcached
	- Advantages
		- Auto Scaling - can be scaled up or down based on growing demands
		- Multi AZ deployment - provides high availability and fault tolerant
		- Simplified management - cloud provider handles setup, maintenance, patching and monitoring, allowing development team to focus only on their application related concerns rather than the infrastructure
		- Flexible Data model - pay as you go model, only pay for the resource being used
	- Disadvantages
		- Dependency on Cloud Provider - vendor lock in
		- Network costs and Latency - data retrieval is fast but n/w overhead is there which leads to additional data costs and latency

### Caching Strategies

#### Read Aside or Lazy Loading
- data is stored in the cache only when it is explicitly requested by the application
- cache doesn't preemptively store the data
- when application sends a read request to cache then two things happen -
- Cache hit - if the data is present in the cache it is retrieved from the cache and returned
- Cache miss - if the data is not present in the cache, then AS makes a request to the database, gets the data and then store it in cache for future use
- Need to set TTL to remove stale data
- Advantages
	- Simple and effective as cache stores only frequently access data
	- Reduces memory usage as only frequently access data is stored, data is not stored upfront
	- System can tolerate cache failures
	- useful for read heavy system with infrequent updates
- Disadvantages
	- cache miss for less frequently accessed items creating cold start
	- data inconsistency is a challenge - if the data is stored in the cache and afterwards it is updated in the database, then app server will read the data from the cache until it is evicted, can be solved using TTL or with some write strategies
-  Use cases
	- Social Media app
	
#### Read Through
- application sends a read request to cache, two things happen -
- Cache hit - if the data is present in the cache it is returned
- Cache miss - if the data is not present in the cache, then cache makes a request to the database, stores the data and returns to the AS
- Need to set TTL to remove stale data
- Advantages
	-  application only needs to communicate with cache, simplifying the application code
	- useful for read heavy system with infrequent updates
	- only frequently access data is stored
- Disadvantages
	- system can not tolerate cache failures as request first goes to cache
	- data consistency issue -> can be solved using TTL or with some write strategies
- Use cases
	- Social Media app

#### Write Through
- application sends a write request to the cache, cache then immediately writes data to the database
- This is a synchronous process as both the cache and the database are updated as part of the same operation
- can be combined with read through - as once the data is written to the cache and the database, it becomes consistent and when the read request comes to the cache it will always be cache hit and the latest data will be returned
- when the write request comes, if the data is present in the cache then it is invalidated and written to the cache and then to the database
- Advantages
	- guarantees strong consistency between cache and the database - data remains consistent, no need to set TTL
	- useful for applications where data is written once and read multiple times as it guarantees strong consistency
- Disadvantages
	- write latency is high as data needs to be written to both the cache and the database
	- Infrequently accessed items are also stored in the cache, TTL can be set to remove those
- Use cases (need stronger consistent guarantees)
	- banking application
	- reservation system

#### Write Around
- application sends a write request directly to the database bypassing the cache
- cache is populated with the data when read request comes to the cache either through cache aside or read through strategies
- Cache hit - If the data is present in the cache, return the data but it might be stale data as it is not invalidated therefore TTL can be used to remove the stale data
- Cache miss - If the data is not present in the cache, it is read from the database, stored in the cache and then returned
- Advantages
	- useful for write heavy applications where data is frequently updated but not immediately read as it will return stale data
	- only frequently access data is stored
- Disadvantages
	- doesn't guarantee strong consistency between cache and database 

#### Write back (Write behind)
- data is written directly to the cache and then it is written to the database asynchronously (periodically in batches)
- can be used with read through caching strategy
- Advantages
	- write latency is low as data is written first only to the cache
	- useful for write heavy application
	- do not need to set TTL as the cache is the source of truth here
- Disadvantages
	- risk of data loss if the cache fails before the data is written to the database - can be mitigated using persistent caching solution
	- inconsistency between cache and the database

### Challenges in Distributed Caching
- Data consistency between cache and the database - can be achieved using some strategies
	- Cache invalidation - set a TTL for a cache entry to invalidate the record
	- Write Through - guarantees strong consistency between cache and the database
	- Event based Updates - whenever any data is written to the database an update event will be triggered to the cache to update or invalidate the data in the cache
	- Leasing - allows only one node to update the cache entry at a time to prevent conflict and maintain consistency between nodes
- Network Partition
	- if there is a n/w partition then cache nodes will operate independently to handle read and write requests and the data will be in inconsistent state
	- can be solved used
		- quorum based consensus - allows only a subset of nodes to operate during n/w partition to avoid conflicting states and requires consensus for updates

### Hot Key Problem
- Let's consider for our newsfeed system we are storing the posts created by users in redis cache
- As we will be having many posts created by users so all the posts will be distributed across multiple cache servers in our redis cluster
- Consider the data is partitioned / sharded across the cache servers based on the userId
- Shard A contains the posts made by a normal user and Shard B contains the posts made by a celebrity user
- Because Shard B contains the data for the celebrity user it will get more load compared to Shard A so there is an uneven distribution of load, this is called hot key problem
- Shard B will continue to get more and more load compared to other shards and eventually it will fail
- How do we solve it ?
- Approach 1 - we can create multiple read replicas of the hot shard (Shard B) and it should be horizontally scalable with load (whenever any request comes for that particular celebrity user, instead of going to only shard B, it can go to multiple read replicas thus reducing the load on shard B)
- Approach 2 - we can have in memory cache in our client so they aren't making so many calls to redis for the same data

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
