- store frequently access data in faster storage like in-memory
- caching improves the data retrieval speed, overall system performance, reduces latency and faster response time
- places where caching can be used
	- Browser
	- CDN
	- API Gateway
	- Services
	- Distributed cache (Redis, Memcached)
	- Database
- Cache Replacement strategies
	- LRU - removes the least recently used items from the cache based on their last accessed timestamp, this approach is suitable for caching hot keys
	- LFU - removes the items from the cache based on their usage count
- Cache Invalidation strategies
	- required to make sure data remains consistent between storage and cache, to refresh the stale data in cache
	- Invalidation when writing
		- when data is modified in the database, it also gets actively invalidated in the cache
		- widely used approach
	- Invalidation when reading
		- data is read from the cache and the validity is checked, timestamp can be used to check the validity of data
		- if the data is stale or not valid then it is read from the database and written to the cache
		- introduces complexity
	- TTL
		- each data has a ttl associated with it
		- if ttl expires, data gets removed from the cache


### Cache reading strategies

#### Cache aside
- known as lazy loading
- application directly communicates with both the cache and the storage system
- application sends a read request to the cache
- if the data is present in the cache it is returned
- else there is a cache miss and the application sends a read request to the storage system
- storage system returns the data to the application
- and then the application sends a write request to write the data into the cache for future reads
- Pros -
	- system can tolerate cache failures as it can still read from the storage
	- cache and storage can have different data models
- Cons -
	- application needs to manage both cache and storage
	- data consistency issue -> can be solved using TTL or some write strategies

#### Read through
- application directly communicates with cache only 
- cache act as an intermediary between application and storage system
- cache is responsible for reading data from the storage system and updating the cache itself
- application sends a read request to the cache
-  if the data is present in the cache it is returned
-  else there is a cache miss and the cache sends out a read request to the storage
- storage system returns the data to the cache
- data is written to the cache and then it is returned to the application
- Pros -
	- application only needs to communicate with cache
	- useful for read heavy system
- Cons -
	- cache and storage system should have same data model
	- system can not tolerate cache failures as request first goes to cache
	- data consistency issue -> can be solved using TTL or some write strategies


### Cache writing strategies

#### Write around
- application writes data directly to the storage system bypassing the cache
- can be combined with cache aside and read through
- application sends a write request to the storage system
- once the data is written to the storage, one of the following steps may be taken
- application sends a write request to the cache
- invalidate the data in the cache then subsequent read request for the same key will result in cache miss and will be read from the storage
- do nothing and wait for the TTL mechanism to invalidate the data
- Pros -
	- simple and easy, decouples the cache and the storage system
- Cons -
	- subsequent read request for the same key will result in cache miss and will be read from the storage
	- If data is written once and read again, storage system will be accessed twice
	- If data is frequently updated and read then storage system will be accessed multiple times 
	- temporary data inconsistency

#### Write through
- application writes data directly to the cache
- cache act as an intermediary between application and storage system
- cache is responsible for writing data to the storage system and updating the cache itself
- can be combined with read through
- application sends a write request to the cache
- cache then sends the write request to the storage system
- once the data is written successfully to the storage system then it is returned to the cache
- data is written to the cache
- Pros -
	- application only needs to send write request to the cache simplifying the process
	- ensures consistency or upto date data in the cache - how ????
		- when a write request comes cache first invalidate the data and then sends the write request to the storage
		- if the write is successful in the storage the it is written into the cache
		- it makes sure that data is either updated or invalidated
- Cons -
	- increases write latency as data needs to be written in two places


#### Write back
- application writes data directly to the cache
- cache act as an intermediary between application and storage system
- data is written to the storage system asynchronously (periodically in batches)
- can be combined with read through
- Pros -
	- reduces the write latency for write heavy application
- Cons -
	- if cache fails then data will be lost
	- temporary inconsistency between cache and the storage


### Cache strategy combinations
- cache aside and write around
- read through and write through
- read through and write back


### Cache inconsistency scenarios
- multiple cache replicas
- writing order of cache and storage
- concurrent cache updates

### Distributed Caching
- data is stored across multiple nodes why ???? - amount of data is very large so it cannot be stored in a single machine, it needs to be distributed across multiple machines or nodes
- helps to achieve scalability, fault tolerance and distributes the load evenly across nodes
- fault tolerance can be achieved using consistent hashing and data replication

#### Challenges in Distributed Caching
- Data Consistency
- Cache invalidation
- Scalability -> can be achieved using Consistent Hashing

