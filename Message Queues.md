- are responsible for communication or transferring of data between different systems
- is a communication mechanism that enables different components of a system to send and receive messages asynchronously
- benefits -
	- fan out
		- producer will push messages to the queue and consumers consume messages at their own pace
	- asynchronous processing
		- in ecommerce order service usually doesn't wait for the payment service to complete the payment
	- rate limiting
	- decoupling - makes the component decouple of each other
	- horizontal scalability
	- message ordering
	- message persistence
	- batch processing


### Use case of message queues
- When you have sudden surge in your requests, a queue buffers them allowing the system to process them at manageable rate without overloading the server or degrading the user experience.
- distribute work evenly across the system without blocking it
- Need to propagate events to multiple components

### Things about Message queues
- FIFO - messages are processed in the same order as they were received
- Retry mechanism - most queues have built in retry mechanisms
- DLQ (Dead Letter Queue) - store messages that cannot be processed, useful for debugging and auditing 
- Scaling with partitions - queues are partitioned across multiple servers to store large amount of data, each partition can be handled by a different set of workers
- Backpressure - reduces the production of messages when queue is full

### Why message queues or messaging systems ?
- E.g - Flash sale in ecommerce
- How large no. of requests coming from Service A can be handled by Service B ?
- How requests can be received from multiple sources ?

### How messages are transferred between systems ?
- #### Queuing
	- producers publish messages to the queue from the rear end and consumers consumes messages from the front of the queue
	- a message can be processed by only one consumer at a time
	- once it is processed it is removed from the queue
	  
- #### Publish Subscribe system
	- messages are published to a particular topic in a queue
	- multiple consumers can subscribe to the topic and gets all the messages published in that topic

## Event Streaming System