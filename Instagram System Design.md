## Requirements
### Functional Requirements
- make a post (by uploading photos, videos)
- posts consists of a text, photos/videos, likes, comments
- share post(photos, videos) with other people
- comment on a post
- like on a post and comment
- follow other users
- news feed of a user should consists of the top posts of all the people the user follows. Each page should display the top 20 posts.

### Non Functional Requirements
- Highly available
- News Feed should have minimum latency (near real time)
- Fault tolerant

## Back of the envelope estimation
- DAU = 100 M
- Each user uploads 2 posts per day
- Total posts per day = 200 M
- Avg size of a photo = 100 KB
- Avg size of a text = 100 KB
- Avg size of a video = 1 MB
- Storage reqd for single post = text(100 KB) + photo(100 KB) + videos(1 MB) + likes(5 KB) + comments(10 KB) = 1215 ~ 1300 KB
- Storage reqd for 200M posts per day = 200 M * 1300 KB = 260 TB
- Storage reqd for posts per year = 260 TB * 30 * 12 = 93600 TB
- each user views their news feed avg 5 times per day
- Total news feed requests per day = 100 M * 5 = 500 M
- Display total of 200 posts in the newsfeed of a user with each page containing 20 posts
- Storage required for newsfeed per user  = 200 posts * 1300 KB = 260MB
- Storage required for newsfeed for all the users = 100 M * 260 MB = 26 PB (can't store all the newsfeed in a single system, need to shard it)


### Database Design
Choose NoSQL DB as we need to store huge data

User Table

| id  | name | email | phone_number | last_active_date | status |
| --- | ---- | ----- | ------------ | ---------------- | ------ |
|     |      |       |              |                  |        |
Follower Followee Table
- who are the followers of user A (userB, userC)
- who userA follows (user D, user E)

| followee_id | follower_id |
| ----------- | ----------- |
| user A      | user B      |
| user A      | user C      |
| user D      | user A      |
| user E      | user A      |
Post Table

| id  | content | photo_id | video_id | user_id | created_at | shared |
| --- | ------- | -------- | -------- | ------- | ---------- | ------ |
|     |         |          |          |         |            |        |
Photo Table

| id  | description | text | path |
| --- | ----------- | ---- | ---- |
|     |             |      |      |
Video Table

| id  | description | text | path |
| --- | ----------- | ---- | ---- |
|     |             |      |      |
Comment Table

| id  | text | post_id | user_id | created_at |
| --- | ---- | ------- | ------- | ---------- |
|     |      |         |         |            |
Like Table
type can be comment or post

| id  | type | user_id | post_id | comment_id | created_at |
| --- | ---- | ------- | ------- | ---------- | ---------- |
|     |      |         |         |            |            |

## API Design
- publishPost(userId, content)
- followUser(follower_id, followee_id)
- postComment(postId, userId, content)
- 
- getNewsFeed(userId)

## High level design

#### Simple Design

- Posts or Feed publishing flow
	- user sends a request to post service via load balancer
	- post service saves the metadata info in the post cache and database
	- media files like photos and videos are stored in the object storage like S3
- Follow other users flow
- News Feed generation flow
	- user sends a request to news feed generation service via load balancer
	- news feed generation service fetches the list of people that the user follows from the Follow service
	- news feed generation service makes a request to the Post service with all the followeeIds and retrieves the top 200 posts made by each of them
	- then it aggregates and sorts them and return the top 200 posts from it and stores them in the newsfeed cache
	- newsfeed cache contains data in the form of <user_id, post_id>
	- separate newsfeed cache for each user as data is huge
	- it's a pull based approach or fanout on read
	- cons of this approach -
		- very slow as news feed is calculated everytime
		- high latency
	- How to improve it ?
		- precompute the newsfeed cache, how ???
		- during feed publish request generate the newsfeed cache using fanout service


#### Improved Design
- Posts or Feed publishing flow
	- user sends a request to post service via load balancer
	- post service saves the metadata info in the post cache and database
	- media files like photos and videos are stored in the object storage like S3
	- push the post to all the followers of the user using Fanout service
	- Fanout service fetches the list of followers of the user from the Follow service then publish the post to the message queue
	- Workers service consumes the post from the message queue and stores them in the newsfeed cache
	- Notification will be sent to all the followers of the user about the new post available in their newsfeed, it is not a good approach but why ????
	- once the notification is sent to the user they make a newsfeed generation request call 
	- the call goes to the news feed generation service and it fetches the newsfeed from the cache
	- Sending notification to all the followers will degrade the performance of the system as system will be under load
	- If the user is a celebrity then they will have millions of followers and sending notification to those followers is not a good idea
	- Maybe you want to send notifications to only some set of followers or their loyal followers or if the user is not a celebrity then send notifications to all its followers
	- Use fanout on write or push model if the user is non-celebrity else use fanout on read or pull model
- News Feed generation flow
	- user sends a request to news feed generation service via load balancer
	- news feed generation service fetches the newsfeed from the newsfeed cache if the request comes from the user after getting a notification (push model)
	- else user (if it follows a celebrity user) requests news feed generation service using pull model after every specified interval or when it comes online or if it refreshes the news feed page
- How Post data will be sharded ?

