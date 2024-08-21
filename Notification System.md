### Functional Requirements
- What type of notifications are we going to support ?
	- Push notifications on devices, SMS and Emails
- Will it be a real time notification ? - Yes
- Type of devices we are going to support - iOS, Android, Laptops
- How many notifications are we going to support ?
	- 1 M push notifications
	- 20 M SMS
	- 50 M Emails

### Non Functional Requirements
- Highly available
- Scalable
- Fault tolerant
- Low latency


### High Level Design

#### Simple Design
- Notification Flow
	- multiple services sends notification to notification service via the load balancer
	- notification service creates the notification request payload to be sent to the third party service providers
	- notification service builds the request payload from the metadata database which contains user metadata info, device info
	- third party service providers delivers the notification to the respective users
- Problems with the above flow
	- Single point of failure as there is only one instance of notification service -> there should be multiple instances of notification service
	- Scalability issue as notification service is doing everything from receiving the requests from multiple services to building the request payload to be sent to the 3rd party services and waiting for them to be delivered it to the users, so notification service and 3rd party services are tightly coupled -> needs to be decoupled, use message queues

#### Improved Design
- send notifications to message queues
- each notification event will be sent to its corresponding message queue
- notification service doesn't need to wait for the response from the 3rd party service providers
- notification events will be processed from the message queues by the corresponding workers
- workers sends out the notifications to the service providers
- service providers send the notifications to user devices
- notification settings table contains the user preference info


### Database Design
- User Table

| user_id | name | phone_no | email |
| ------- | ---- | -------- | ----- |
|         |      |          |       |

- Device Table

| device_id | device_token | user_id |
| --------- | ------------ | ------- |
|           |              |         |

- Notification Settings Table

| id  | notification_type | opt_in |
| --- | ----------------- | ------ |
|     |                   |        |


### Deep Dive
- How will you ensure reliability ?
	- store all the notifications in the database
	- what database to use ? (NoSQL)
- How to handle errors ? 
	- notification events will be sent back to the queue
- How will you ensure that notification service is not overwhelmed by too many notifications at a time ? Rate Limiter

