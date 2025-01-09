
## Change Feed
- It is a persistent record of changes to a container in the order they occur
- It works by listening to a azure cosmos container for any changes
- It outputs the list of documents that were changed in the order in which they were modified
- The output is distributed across multiple consumers for parallel processing
- It enables you to build efficient and scalable solution for these type of design patterns
	- Event computing and notification
		- change feed triggers notification or make an API call based on certain events such as creation, updation and deletion
	- Real time Stream processing
		- allows real time stream processing for IoT or real time analytics processing on operational data
	- Data movement
		- read from change feed for real time data movement

## Change Feed Modes
- Change Feed offers two type of modes
- Change feed can be consumed in different modes across different applications for the same Azure cosmos db container
	- Latest version mode
		- It is a persistent record of changes made to items from insert and update operations
		- stores only the latest change or version of each item in a container
		- doesn't capture the delete events and intermediate events
		- If you delete an item from your container, it's also removed from the change feed.
		- Use soft delete marker to delete an item and it will appear in the change feed
		- If an item is created and then updated before you read the change feed, then only the updated version appears in the change feed
		- changes are always available as there is no fixed data retention period, As long as an item exists in your container, it's available in the change feed.
		- starting point to read change feed can be from the beginning of the container, from a point in time, from "now," or from a specific checkpoint.
	- All versions and delete mode (preview)
		- It is a persistent record of all changes made to items from insert, updates and deletes
		- stores the record of each change to items in the order in which they have occurred
		- If an item is created and then updated before you read the change feed, then both the create and the update version appears in the change feed
		- captures the delete and intermediate events, also captures the delete from TTL expiration
		- Deletes from TTL expirations aren't guaranteed to appear in the feed immediately after the item expires. They appear when the item is purged from the container.
		- To read the change feed using this mode, set the continuous backup retention period for your account, e.g - if your container was created eight days ago and your continuous backup retention period is seven days, then you can only read changes from the last seven days.
		- change feed starting point can be from "now" or from a specific checkpoint within the retention period. You can't read changes from the beginning of the container or from a specific point in time by using this mode.


## Change Feed Read options
- There are different ways to read data in change feed
- Push Model
- 
- Pull Model