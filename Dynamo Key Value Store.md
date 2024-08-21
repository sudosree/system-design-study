## 1. Requirements
### Functional Requirements
- support two APIs - get(key) and put(key, context, object)
- get(key) -> 
	- finds the node where the object associated with the key is stored and returns a single object or list of objects with the conflicting versions along with the context
	- context contains the encoded metadata information of the object and the version of the object
- put(key, context, object) ->
	- finds the node where the object associated with the key should be stored
	- context information is stored with the object
	- sends the write request along with the context that it received during the previous read request

### Non-Functional Requirements
- Highly available distributed data store 
- Highly scalable, reliable and durable
- Supports network partition
- No single point of failure (resilient) as data is replicated
- Low latency for read and write operations
- Eventual Consistency
- Leaderless replication is used
- Consistent hashing is used for partitioning the data
- Consistency among replicas are maintained using quorum technique
- Highly available for writes -> leads to conflicting concurrent write issues
- Detect concurrent writes using "happen before relationship" strategy (using vector clocks)
- Resolve concurrent writes -> using merging the conflicting version (CRDTs) or last write wins strategy
- Internode communication and failure detection using Gossip Protocol
- Handle temporary failure using Sloppy quorum and Hinted Handoff
- Handle permanent failure using Merkle Trees

## 2. High Level Design
### How get and put operations work
- Two approaches to send read and write requests -
	- Load Balancer 
		- can route the request to any nodes in the dynamo ring
		- if the node is not a coordinator node then it needs to forward the request to the coordinator node for a particular key say K (additional hop)
		- The node can forward the request to the designated coordinator node for the key K by referring to its membership state information which contains which nodes are reachable and the preference list for any key
	- Partition aware client library
		- client uses this library
		- client periodically picks a random node from the ring and download its membership state information and from this the client knows to which node (coordinator node) request needs to send
		- route the request to the coordinator node for a particular key say K
		- coordinator node is the first node among the top N nodes in the preference list
- requests should always be sent to the top N healthy nodes in the preference list
- dynamo handles temporary failure using sloppy quorum and hinted handoff

#### Put operation using Partition aware client library (when all the nodes are available)
- request comes to the Partition aware client library
- generates the hash of the key say K
- library knows the preference list of key K and routes the request to the coordinator node among the top N nodes in the preference list
- When request comes to the coordinator node it generates a new version along with a new vector clock
- Writes the new version locally 
- Coordinator nodes sends the request to the remaining N-1 nodes from its preference list
- waits for the response from W-1 nodes
- Coordinator node can be the node that replied fastest in the previous read operation (Read your writes consistency) -> always get the data

#### Get operation using Partition aware client library (when all the nodes are available)
- request comes to the Partition aware client library
- generates the hash of the key say K
- library knows the preference list of key K and routes the request to the coordinator node among the top N nodes in the preference list
- request comes to the coordinator node it gets a version
- coordinator node sends the request to the remaining N-1 node and request their versions of the same data
- If coordinator gets multiple conflicting versions (not causally related) then it sends all the versions to the client
- client resolves the conflicting versions using merging techniques and writes back the data
- After the read request is completed system waits for some time and if any stale responses are received then coordinator node updates the stale data with the new data in those nodes -> Read Repair

### How consistency is maintained when nodes become available
- Read Repair
- Anti Entropy process

### How does Dynamo achieves high availability, fault tolerance, durability, reliability and scalability
- makes the system fault tolerant and guarantees durability and reliability using replication -> leaderless replication
- makes the system highly available writable data store -> using sloppy quorum and hinted handoff
- even in case of n/w partition or server failures, it should accept write requests
- leads to conflicting concurrent write issues as multiple nodes can accept write requests
- scalability is achieved using consistent hashing with virtual nodes

### How conflicting concurrent write issues are resolved using vector clock
- conflicts are detected using vector clocks 
- when conflicts are resolved -> conflicts are resolved using merging different versions of the same data during read operations (e.g - CRDTs)
- who done the conflict resolutions -> conflicts resolutions can be done either in server side (datastore) or in client side (application)
- If conflict resolution is done by datastore then LWW (Last Write Wins strategy) can be used -> not a good choice, can lead to data loss
-  If conflict resolution is done by the application then different versions are merged using different techniques (CRDTs Conflict free replicated datatypes -> good choice, no loss of data)
- Multiple conflicting versions of the same object occurs during n/w partition with concurrent writes
- Two versions of an object are causally related to each other if D1([A:2][B:0]) and D2([A:2][B:1]) i.e. the counters of all the nodes in the 1st version are less than or equal to the counters of all the nodes in the 2nd version
- Two versions of an object are concurrent if D1([A:3][B:0]) and D2([A:2][B:1]) i.e. if the counter of any nodes in the 1st version is more than its corresponding counter in the 2nd version

### How replication works in Dynamo
- data should be replicated to N nodes (N = replication factor)
- uses quorum like techniques -> R+W > N 
- To achieve low latency on read and write R < N and W < N
- dynamo uses (3,2,2) as (N,R,W)
- coordinator node receives the request and sends the request to the remaining N-1 nodes in the preference list for a key
- Preference list always contains more than N nodes to handle failures
- Requests are always processed by the top N healthy nodes in the preference list
- Because of virtual nodes preference list should always contains distinct physical nodes
- coordinator node can be any of the node among the top N nodes in the  preference list to handle load distribution

### Disadvantages of basic consistent hashing
- Partition size of each node might not be the same due to node addition or removal
- Random node distribution on the ring leads to non-uniform data and load distribution

### Advantages of using virtual nodes over basic consistent hashing
- Data and loads are uniformly distributed when nodes are added or removed
- Each node is assigned to multiple positions on the ring which leads to uniform distribution of data

### How sloppy quorum and hinted handoff works to handle temporary failures and achieve high avaialability
- Sloppy Quorum
	- If W or R nodes among the N nodes are not reachable due to n/w partition then read an write requests are still allowed and goes to R and W replicas that are available but not part of N nodes
- Hinted Handoff
	- When a replica A is not available for write request, request goes to another replica B and in B there will be hinted replica in its metadata information which tells the intended recipient 
	- When A becomes available, data is transferred from B to A and then deleted from B

###  How permanent failures are handled
- Anti entropy using Merkle Trees
- anti entropy means replica synchronization protocol to keep the replicas in sync 
- Each node maintains its own separate Merkle Trees for its every key range
- Leaf nodes are the hashes of data associated with the key
- Parent nodes are the hashes of their children
- If the hashes of the root of two trees are same then both replicas are in sync for the same key range
- Else the replicas are out of sync
- Advantages -
	- each branch can be checked independently of other branch
	- data movement can be reduced
- Disadvantages -
	- key ranges can be changed due to addition or removal of nodes which leads to recalculation of Merkle Trees

### How failures are detected using Gossip Protocol
- Gossip Protocol is used to get the state information of every nodes i.e. which nodes are available and what key ranges are served by which nodes
- peer to peer commnunication protocol
- every node keep track of the membership list (status of every other nodes whether they are alive or not) by sending its state information along with the state information of other nodes to some other random node
- failures are detected when a node has stopped sending out heartbeats to other nodes

