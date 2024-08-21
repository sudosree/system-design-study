- are responsible for communication or transferring of data between different systems
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