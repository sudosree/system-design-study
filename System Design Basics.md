
### Scalability
- ability of a system to handle the increased workload without losing performance
- Increased workload means - increase in user base, data volume, features
- Different ways to achieve scalability
	- Vertical Scaling
	- Horizontal Scaling
	- Load Balancing
	- CDN
	- Caching
	- Database Sharding
	- Asynchronous communication (loose coupling)
	- Microservices architecture

### Availability
- system continues to operate or function and responds to requests despite failures (even if some components are unavailable), ensuring that every request must receives a response either success or failure
- Different strategies to achieve high availability
	- Redundancy
		- having backup component, if the primary component fails the backup component takes its place
		- helps to achieve reliability, availability and fault tolerant
		- Different architectures of redundancy
			- Hot Cold
				- All the read and write request goes to the hot replica
				- cold replica is in standby mode, when hot replica becomes unavailable then cold replica will take its place
				- cons - waste of resources as cold replica is in standby mode most of the time
			- Hot Warm
				- All the write requests goes to the hot replica and read requests goes to the warm replica
				- cons - read requests might return stale data
			- Hot Hot
				- both the replicas accept the write requests
				- cons - lead to write conflicts during replication
	- Replication
	- Load Balancing
	- Rate Limiting
		- restrict the no. of requests to the server to protect it against DDOS attacks
	- Service Degradation
		- provide only core services and remove non essential ones
	- Queuing
		- unlike rate limiting it allows the requests to go into the server but makes them wait until the previous one finishes, it increases the latency
	- Circuit Breaking
		- prevents cascading failure in distributed system

### CAP Theorem
- Consistency, Availability and Partition Tolerance
- It is impossible for a distributed system to guarantee all the three at the same time
- Consistency - every read request must receive the latest write data
- Availability - every request must receive a response despite n/w failures even if some read requests receive the stale data 
- Partition Tolerance - system continues to operate despite n/w failures where nodes can't communicate with each other
- CP and AP systems are possible
- CP systems - e.g - Google's BigTable
- AP systems - e.g - Amazon DynamoDB, Cassandra

### Consistency Patterns
- Strong Consistency
	- every read requests on servers must receive the most recent upto date write data
	- replicates the data using leader follower synchronous replication strategy
	- Pros
		- consistent data view across the system
		- highly reliable and durable
	- Cons
		- reduced availability
		- low latency
		- resource intensive
	- Examples
		- Finance systems like banking
		- Relational databases (MySQL, PostgreSQL)
		- File systems
		- Non Relational databases (Google's BigTable)
- Eventual Consistency
	- when a write request is executed against a server, immediate subsequent read requests against other servers do not return the latest write data
	- the system will eventually converge to the latest state
	- replicates the data using multi leader or leaderless replication strategies
	- Pros
		- Highly available
		- scalable
		- low latency
	- Cons
		- data inconsistency
		- potential data loss
		- data conflicts
		- weak consistency model
	- Examples
		- DNS
		- SMTP
		- S3
		- Retail application
		- Comments or posts on social media platform like Facebook
		- Non Relational Database - Amazon DynamoDB, Cassandra
- Weak Consistency
	- when a write request is executed against a server, subsequent read requests against other servers may or may not return the latest write data
	- can be achieved using write back cache pattern
	- write request is received by the cache, cache sends the write request to message queue and then it sends the acknowledgement, write requests then written to the database asynchronously from the message queue
	- Pros
		- Highly available
		- low latency
	- Cons
		- data inconsistency
		- potential data loss
		- data conflicts
		- Not reliable and durable
	- Examples
		- Live streaming 
		- multi player video games


### Consistent Hashing
