- multiple concurrent write requests happens in multi leader and leaderless replication
- multiple leaders accepts the write requests and conflicts will arise when data are asynchronously replicated to other leaders
- Two write requests are concurrent if both are happening at the same time or even if they are happening at some interval gap but none of them is aware of the other
- Two write requests are causal if both of them knows about each other
- Detect concurrent and causal requests using Happen Before Relationship using version numbers

### Detect concurrent write requests
- using happen before relationship
- server will always read the data from the database before it makes a write request
- read request from the database will return the current version no and the list of all concurrent values (that have not been merged yet)
- when server sends a write request, it sends the version no that it received from the previous read request and the data it wants to update to, to the database
- database checks if the version no. it receives from the write request is less than or equal to or greater than the current version no. it has
- If the version no. is less than the current version no then it is a concurrent request, database keeps both the values and increment the version no.
- every write request increments the version no.
- If the version no. is equal to or greater than the current version then it's a causal request then the database overwrites the previous data with the new data and increment the version no.
### Capture Happen Before Relationship
- used to detect write conflicts - concurrent and causal requests
- when two requests are causal then overwrite the first request with the second request 
	- when a database receives a write request with a version no. greater than or equal to it's current version no. then it overwrites all the values of the current version and the lower version number with the current value
- when two requests are concurrent then keep both the values do not overwrite it, either database can merge it and give it to the server or the database returns all the concurrent values to the server and it is upto the server to merge the concurrent values and return it to the database
	- when a database receives a write request with a version no. smaller than it's current version no. then it keeps both the values as this is concurrent request as the new request is unaware of the current state of the database as it is based on the previous state of the database

### Resolve write conflicts
- Last Write Win Strategy
	- assign a unique identifier to write requests say timestamp or UUID
	- whichever requests has the bigger timestamp accept it and discard the other
	- loss of data - not durable
- Merge the concurrent values 
	- automatically merge the conflicting values or concurrent or sibling values
	- there are many automatic conflict resolution methods - 
		- Conflict Free Replicated Data Types (CRDTs)
		- Operational transformation
- The above two strategies work for multi leader and leaderless replication
- For single leader replication - accept the first request and fail the other, it can be done using
	- Pessimistic locking
		- prevents users from concurrently updating the same record by applying locks
		- Pros
			- prevents application from updating the data that is being changed or has been changed
			- useful when data contention is high
		- Cons
			- introduces deadlock as multiple resources are blocked
			- not scalable because if a transaction has been locked for long time it degrades the database performance
	- Optimistic locking
		- allows users from concurrently updating the same record 
		- doesn't use locks
		- Pros
			- use version vector to implement it
			- prevents application from updating the stale data
			- no deadlock
			- useful when data contention is low
		- Cons
			- not useful when data contention is high


### Version Vectors
- collection of version no.s of all replicas
- When concurrent writes happens on multiple replicas, keeping only version number for each key will not work. You need to use version number of every replica as well as for each key.
- Each replica updates the version number of a key whenever data is written to that key and also keep track of the version number of the other replicas it has seen so far
- When a client reads it receives version vectors from the database
- When a client writes, it needs to send the version vectors to the database
- [2,0] {milk, flour} and [0,1] {eggs} are concurrent requests because the version no. of Database1 in the version vector of Database2 is less than the version no. of Database1 in the version vector of Database1 which means the Database2 is unaware of the current state of Database1, so after replication both data will remain
- [2,1] (A - {milk,flour}, B - {eggs}) and [2,2] ({milk,flour,eggs,bread}) are causal requests because the Database2 is aware of the current state of itself and Database1
	