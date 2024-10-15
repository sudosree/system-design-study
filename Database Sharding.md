
- splitting a large database into multiple smaller and manageable shards
- improves scalability, query performance and availability
- Types of sharding - 
- Range Based Sharding
	- database rows are split based on range of values across multiple shards
	- no. of shards are fixed
	- cons - data distribution may not be uniform
- Modulus Sharding
	-  assigns a key to a particular shard based on the hash value of the key modulo the total no. of shards
	- cons - when shards are added or removed, keys needs to be re-distributed again
- Key or Hash Based Sharding
	- assigns a key to a particular shard using hash function
	- data distribution is uniform
	- can be achieved using Consistent Hashing
	- cons - cannot perform efficient range queries
- Directory Based Sharding
	- query the lookup table to get the shard for a particular data
	- cons - any failures to the lookup table will affect the availability of the database
- challenges with sharding
	- complexity
	- cross shard joins
	- data rebalancing

### Routing strategies in a sharded database
- Shard aware node
	- client sends a request to a node, if the node owns a shard relevant to the request it is handled else it routes the request to a node which owns the shard for the request
- Routing Tier
	- routing tier handles all the request, it sends the request to the dedicated node which contains the data in the shard relevant to the request
- Shard aware client
	- client contains the information about all the nodes and the mapping of shard to node
	- clients get this information from the configuration service


### Database scaling strategies


## CDN (Content Delivery Network)
- refers to geographically distributed servers called edge servers
- provides fast delivery of static content
- Benefits
	- Improved load time
	- Increased content availability and redundancy
	- scalability