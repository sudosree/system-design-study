- Kafka is a distributed event streaming platform that can be used either as a message queue or as a stream processing system
- Kafka is highly performant, scalable, durable, reliable and highly available
- Kafka cluster is made up of multiple brokers. Brokers are called servers. 
- Broker is a server. It is responsible for storing data and serving data to client.
- Broker consists of multiple partitions. 
- A partition is an ordered immutable sequence of messages that is continuously appended at the end of a log file
- Partitions are the way for Kafka to scale as they allow messages to be consumed in parallel
- Topic is a logical grouping of partitions. It's a way to publish and subscribe data in Kafka. 
- When you publish message, you publish it to a topic and when consume message you consume it from a topic.
- A topic can have multiple producers
- Topic is a logical grouping of messages and a Partition is a physical grouping of messages.
- A topic can have multiple partitions and each partition can belong to a different broker.
- Topic is a way to group data and partition is a way to scale data
- Producer produces messages and Consumer consumes messages to/from a topic
- Kafka can be used both as a Message Queue and Event streaming platform. The difference is - Consumers read messages from the queue and acknowledge it once they process the message. In case of stream processing, consumers read messages, process them but do not acknowledge them. This allows for more complex processing of the data.
- Consumer Group is a group of consumers, each message or event can be processed by only one consumer in a consumer group

### How Kafka Works
- Producer sends messages to a topic in kafka
- Messages are also called records
- Each message consists of one required file - the value and three optional fields - key, timestamp and headers
- Key is used to determine the partition of the message, which partition the message is sent to
- Timestamp is used to order messages within a partition
- Headers like http headers are key value pairs can be used to store metadata of message
- When a message is published to topic, Kafka determines the partition for the message
- It's a two step process -
- Partition Selection
	- Kafka uses a hashing algorithm to hash the message key to determine the partition of the message
	- If the message key is not specified then round robin method is used to determine the partition of the message
	- or partition can be determined using a partitioning logic defined in the producer config file
- Broker Assignment
	- once the partition is determined then we need to figure out which broker hosts that particular partition
	- partition to broker mapping is managed by kafka cluster metadata
	- producer uses this metadata to send the message directly to the broker that hosts the target partition
- Each message in a partition is assigned a unique offset (sequential identifier) to keep track of the message's position in a partition
- This offset is used by consumers to keep track of their progress in reading messages 
- There are two ways a consumer consumes messages from a topic
	- Push based model - consumer subscribed to a topic whenever any new messages are available in the topic it is delivered to the consumer
	- Pull based model - consumer polls the topic for any new messages at a regular interval


### Benefits of append only log mechanism in Partition
- Partition is an append only log file
- Messages are sequentially appended to the end of the log file
- Benefits of this are -
- Immutability - once a message is written to a partition it cannot be altered or deleted. It improves the performance and reliability, also simplifies replication
- Efficiency - minimizes disk seek times
- Scalability - Facilitates horizontal scaling, partitions can be scaled across multiple servers to handle the increasing load and can be replicated across multiple servers to handle the fault tolerance


### How replication works in Kafka
- Kafka replicates data in multiple replicas of partition across multiple servers/brokers to handle durability and availability
- There are multiple replicas of a partition across different brokers to handle scalability, durability and availability
- Kafka follows leader follower replication strategy, it works as follows
	- Leader Replica - 
		- each partition has one leader replica and multiple follower replicas
		- leader replica handles all the write and read requests from the client and then send to follower replicas
		- leader replica is assigned by cluster controller
	- Follower Replica
		- each partition has multiple follower replicas across multiple brokers
		- they do not handle any direct requests from the client
		- they asynchronously replicate data among themselves
	- Replication
		- data is asynchronously replicated to follower replicas
		- follower replica tries to keep in sync with the leader replica
		- when leader replica dies then a most upto date follower replica is assigned to a leader replica
	- Cluster Controller
		- responsible for leader replica election
		- when a leader replica dies, it promotes an upto date follower replica to be a new leader replica


### When to use Kafka as a message queue and as a streaming platform
- Kafka can be used both as a message queue and as a streaming platform
- Key difference between the two lies how a consumer consumes the data
- In a message queue, consumers pull messages from the queue when they are ready to process them but in a event stream platform, consumers consumes and process messages as in when they arrive in real time
- Use kafka as a message queue under following scenarios -
	- Asynchronous Processing - When data needs to be processed asynchronously
	- Message Ordering - When you need to maintain the order of messages (e.g - Design Ticketmaster)
	- Decoupling - When you want to decouple producers and consumers and both can be scaled independently
- Use kafka as a streaming platform under the following scenarios -
	- When data needs to be processed continuously and immediately as in when they arrive like in real time
	- Messages needs to be processed by multiple consumers simultaneously