### Functional Requirements
- What kind of chat messages are we going to support ?
	- one on one and group based chat
	- Max no. of people in group based chat - 200
- Multiple device support - both web and mobile based
- online and offline statuses
- Is multi-media (images, audios, videos, attachments) supports are allowed ?
	- No only text based messages are allowed
- Limit on text based messages - < 1000 characters
- Push notifications should be sent to user in case if they are offline
- chat messages should be stored forever
- no encryption required
- sent, delivery and read receipts for one on one chats

### Non Functional Requirements
- minimum latency - real time chat experience
- highly consistent - user should see the same chat history on all of their devices
- lower availability can be tolerated in the interest of consistency

### Back of the envelope estimation
- DAU = 100 million
- on average each user will send 20 messages per day
- no. of messages sent per day = 100 million * 20 = 2 billion
- storage required per day = 2 billion * 1000 = 2 TB
- storage required per year = 2 TB * 30 * 12 = 720 TB
- storage required for 10 years = 720 TB * 10 = 7200 TB
- user information, message metadata needs to be stored

### API Design
- sendMessage(message_from, message_to, content)
- receiveMessage(message_from, message_to, content)
- setStatus(user_id, status) - status can be online/offline
- getStatus(user_id)
- sendGroupMessage(message_from, group_id, content)
- getUsersInGroup(group_id)
- joinGroup(group_id, user_id)
- leaveGroup(group_id, user_id)
- getAllFriends(user_id)

### Database Design
- User Table

| user_id | name | phone_number | email |
| ------- | ---- | ------------ | ----- |
|         |      |              |       |
- Message Table (Key value store)

| message_id | message_from | message_to | content | created_at | sent_at | delivery_at | read_at |
| ---------- | ------------ | ---------- | ------- | ---------- | ------- | ----------- | ------- |
|            |              |            |         |            |         |             |         |
- Group Message Table (Key value store)

| group_id | message_id | message_from | content | created_at |
| -------- | ---------- | ------------ | ------- | ---------- |
|          |            |              |         |            |
- User Status Table (Key value store)

| user_id | status | last_seen_at |
| ------- | ------ | ------------ |
|         |        |              |
- Group Table

| group_id | group_name | group_description |
| -------- | ---------- | ----------------- |
|          |            |                   |

- Group User Table

| group_id | user_id |
| -------- | ------- |
|          |         |
- User Chat Server Info Table

| user_id | chat_server_id |
| ------- | -------------- |
|         |                |

### High level design
#### One one one chat messages flow
- user send messages to another user using a chat service
- they will use websocket to send and receive messages to and from chat service
- user first sends a request to load balancer
- load balancer sends the request to service discovery
- all the chat servers registers themselves with the service discovery
- service discovery returns the best chat server for the user and save the mapping in the database (user_id, chat_server)
- once user gets the chat server info, it establishes a websocket connection with the chat server
- user A establishes a websocket (ws) connection to the chat server 1
- user B establishes a websocket (ws) connection to the chat server 2
- each chat server maintains user to connection object mapping in a key value table
- user A sends a message to CS 1 for user B, chat server 1 saves the message to the database and also send it to the message queue (so that it can be sent whenever it is possible)
- how CS 1 knows that CS 2 is the intended chat server for user B ?
	- CS 1 asks the service discovery about the chat server for user B
	- service discovery returns the CS 2 info
- CS 2 will retrieve messages from the message queue and checks the intended recipient of the messages
- If the recipient/user B maintains a ws connection to the chat server 2 then we need to check if the user B is online or offline
- status of user B can be find by the Presence servers, it returns the status of a user
- If user B is online then CS 2 delivers the message else if the user B is offline then CS 2 sends the message to the notification service which in turn sends push notification to the user B

#### How sent, delivery and read receipts are sent to the sender 
- when CS 1 stores the message in the database then we are sure that message will be sent to user B whenever it's possible, it won't be lost
- CS 1 sent a sent receipt to user A
- when user B receives the message from CS 2 it sends an acknowledgement to CS 2 that it received the message
- CS 2 needs to send this ack to CS 1 but how it will know that it is CS 1 ?
- CS 2 gets the message_from info from the message metadata and asks the service discovery about the intended chat server for user A
- service discovery returns the CS 1 info to the CS 2
- service discovery sent the delivery receipt to CS 1 and CS 1 sends it to user A
- same flow is for read receipt

#### How online and offline statuses are maintained
- Presence servers are responsible for maintaining the statuses of the users
- users sends a heartbeat event to the presence server periodically say 3 sec
- when presence server doesn't receive any heartbeat event from the user for x secs then it considers the user as offline and mark the status to offline
- when status is marked online, last_seen_at will also be updated based on the user activity (send/receive message)

#### How a user know the statuses of other users
- there are two mechanisms - push and pull
- push model
	- presence server uses publish subscribe model
	- when user A status changes, then status info is stored in the key value database and presence server publishes the status info to the message queue so that the corresponding users or friends of user A who are subscribed to the message queue can pull the status info of user A from it
	- this is good choice if the friends list of a user is not too big or if a group contains min no. of users
- pull model
	- user B will fetch the status of user A from the presence server whenever user B opens the app or starts a chat with user A


#### Group chat messages flow
- see the diagram