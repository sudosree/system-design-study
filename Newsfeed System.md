### Functional Requirements
- User should be able to publish any posts or feeds and can view their newsfeed
- How the newsfeed will be generated ? - based on the posts, status, images, videos shared by the people that a user follows
- Feeds should contain text, images, videos
- How the newsfeed will be displayed ? - reverse chronological order

### Non Functional Requirements
- Minimum latency while fetching newsfeed
- Highly available
- Fault Tolerant
- Scalable

### Back of the envelope estimation
- User can have 300 friends
- DAU - 300 million
- each user view their newsfeed on avg 5 times a day
- Newsfeed requests per day = 300 million * 5 = 1500 million
- How many posts should be there in a newsfeed ? - 500
- Size of each post - 1 KB
- Storage required for posts in a timeline per user - 500 KB
- Storage required for posts in a timeline for all the active users - 500 KB * 300 million = 150 TB

### System APIs
- Feed Publishing API
	- when a user publishes any feed it should be visible in the newsfeed of his/her followers
	- POST request - publishFeed(userId, content) - /v1/feed
- NewsFeed Building API
	- newsfeed should be created with the aggregation of all posts made by his/her friends the user is following in reverse chronological order
	- GET request - retrieveFeed(userId) - /v1/feed


### Database Design

- User Table

| id  | name | email | phone_number |
| --- | ---- | ----- | ------------ |
|     |      |       |              |
- User Follower Table

| user_id | follower_id |
| ------- | ----------- |
|         |             |

- User Followee Table

| user_id | followee_id |
| ------- | ----------- |
|         |             |
- Post Table or Feed Table

| id  | user_id | content |
| --- | ------- | ------- |
|     |         |         |
- Media Table

| id  | description | path |
| --- | ----------- | ---- |
|     |             |      |

Post Media or Feed Media Table

| id  | post_id | media_id |
| --- | ------- | -------- |
|     |         |          |

### High Level Design
#### Feed Publishing Flow
- user sends a feed publishing request
- request goes to load balancer and from the LB it goes to the application servers
- from the app server it goes to the Post service and the post is saved in Post cache and database
- the feed publish request also goes to the Fanout service and the Notification service
- Fanout service makes the post available to the newsfeed of all the friends or followers of the user
- Notification service notify all the friends or followers of the user when any new post is available

#### NewsFeed Generation Flow
- user sends a newsfeed generation request
- request goes to load balancer and from the LB it goes to the application servers
- from the app server it goes to the NewsFeed service
- NewsFeed service retrieves the newsfeed from the cache (NewsFeed cache)


### Design deep dive
#### Feed Publishing Flow
- When a user publishes any post it can be made available to the newsfeed of its friends or followers in three ways -
- When a user sends a feed publish request, system checks if the user is a celebrity or not
- If the user is a celebrity then its post will not be available to its followers immediately during this flow else the post will be available
- Fanout on write (Push model)
	- Fanout Service fetches the list of friends or followers ids from the graph database for the user
	- Then it fetches the user info from the user cache or the database based on the ids
	- <user_id, post_id> will be sent to the message queue
	- fanout workers pull the data <user_id, post_id> from the message queue and store it in the NewsFeed cache
	- useful for non-celebrity users
	- Pros
		- NewsFeed generation is realtime and precomputed
		- NewsFeed retrieval is fast
	- Cons
		- NewsFeed generation takes time as it retrieves all the friends or followers list and generates the NewsFeed cache (HotKey problem)
		- For inactive users newsfeed generation is a waste of compute resource
- Fanout on read (Pull model)
- Hybrid model
	- Pull model for followers of celebrity users
	- Push model for followers of non-celebrity users

#### NewsFeed Generation Flow
- When a user sends a newsfeed generation request, system checks if the user is a follower of any celebrity
- If yes then Pull model approach will be used else the NewsFeed will be already generated, fetch it from the NewsFeed cache
- Fanout on read (Pull model)
	- applicable during NewsFeed retrieval flow
	- useful for celebrity users
	- NewsFeed service fetches the list of friends or followee ids from the graph database for the user
	- Then it fetches the user info from the user cache or the database based on the ids
	- Then it fetches the top 20 posts (latest, relevant) for each friends from the post cache or the post database , aggregate them and return them in reverse chronological order
	- These posts are stored in NewsFeed cache in the format <user_id, post_id>
	- Pros
		- Good for inactive users who rarely login so no waste of compute resource
		- NewsFeed generation doesn't take much time as no hotkey problem
	- Cons
		- NewsFeed retrieval is slow as it is not precomputed
- NewsFeed cache can be refreshed after some fixed interval every day
- Notification will be sent to users if any new posts are available so that the user can send a pull request to refresh its NewsFeed (Pull Model)
- NewsFeed cache contains information in the following way

| post_id | user_id |
| ------- | ------- |
| post_id | user_id |

#### How many feeds should be stored in the NewsFeed cache for a user
- first store top 20 posts of a user
- and as the user scrolls start adding another 20 posts in the cache till all the 10 pages are covered
- Total 200 feeds for a user in the NewsFeed cache

#### How are we going to store the feeds of all the users in the NewsFeed cache
- LRU cache
- If some feeds are not accessed for quite some time those will be removed

#### Sharding
- Based on userId to retrieve the user related info faster e.g - post data, user data
