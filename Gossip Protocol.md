- peer to peer communication protocol based on peer to peer state management service
- maintains the system state by keeping track of how many nodes are active and inactive
- application data and node metadata should be transmitted among nodes
- Highly available, scalable, reliable
- No single point of failure
- Eventual consistency
- Decentralized

## Different message broadcasting techniques
- Point to point broadcast
	- producer sends messages directly to the consumers
	- messages will be lost if both the producer and consumer fail simultaneously
- Eager reliable broadcast
	- every node broadcasts messages to each and every node via a reliable network link
	- messages are not lost
	- Fault tolerant (NSPOF)
	- drawbacks -
		- significant network bandwidth usage due to O(n²) messages being broadcast for n number of nodes
		- sending node can become a bottleneck due to O(n) linear broadcast
		- every node stores the list of all the nodes in the system causing increased storage costs
- Gossip protocol
	- peer to peer decentralized communication protocol
	- every node periodically sends out a message to a subset of random nodes and eventually all the nodes will receive the message
	- use cases -
		- maintains node membership list
		- achieve consensus
		- temporary fault detection
		- database replication
		- leader election
	- characteristics -
		- limits the no. of messages broadcasted by each node
		- limits the bandwidth consumption to prevent the degradation of application performance
		- can tolerate network and node failures

## Types of Gossip protocol
- Anti Entropy Gossip protocol
	- reduces the entropy between replicas
	- replicas are compared and the differences between them are patched
	- nodes with newest data share it with other nodes in every gossip round
	- increases the bandwidth usage as the whole dataset is transferred. To reduce it Merkle tree is used
- Rumor Mongering Gossip protocol
- Aggregation Gossip protocol

## Strategies to spread messages
- Push model
	- a node broadcasts new messages to a subset of random nodes
	- useful when there are few update messages and few nodes in the system
- Pull model
	- a node actively polls from a subset of random nodes for new messages
	- useful when there are many update messages and more nodes in the system
- Push Pull model
	- The push approach is efficient during the initial phase when there are only a very few nodes with update messages.
	- The pull approach is efficient during the final phase when there are numerous nodes with many update messages.


## Gossip Protocol algorithm
- every node maintains a list of subset of random nodes and their metadata
- gossip to a random live node's endpoint periodically
- every node inspects the received gossip message to merge the highest version number to the local dataset