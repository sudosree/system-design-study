- It is a technique that controls the rate at which users or services can access a resource ( can be an API or service )
- Benefits of Rate Limiter -
	- Prevents resource starvation / prevents system abuse / protects system resources - prevents DOS attacks
		- prevents the DOS attack by blocking or rejecting the requests coming from a specific IP address or from a user exceeding the request threshold or from a authentication token
	- Reduce costs - by preventing overuse of resources as it may lead to additional use of resources which incur additional costs
	- Prevents overloading of server
		- blocks the request from a malicious bot or user
		- blocks the requests from a legitimate user if it exceeds the threshold


### Applications or Use cases of Rate Limiter
- Prevents DOS attacks - External rate limiter (Application Level Rate Limiter)
	- prevents the DOS attack by blocking or rejecting the requests coming from a specific IP address or from a user exceeding the request threshold or from a authentication token
- Gracefully handle the surge of users -  External rate limiter (User Level Rate Limiter)
	- traffic pattern is similar to the above usecase
	- all the requests are coming from the legitimate users and not from the attackers
	- allows the requests for some users and rejects the requests for others based on the threshold
	- e.g - social media platform - if a user has exceeded its limit to make a post per hour then drop all its future requests 
- Multi tiered limits or User account level - Internal rate limiter
	- offering different usage limits for different tier
	- limiting the resources to users based on their selected tier
- API level rate limiting
	- to protect the overuse of APIs provided by the third party service providers
	- limits the no. of API calls per user per minute


### Rate Limiting Response
- fall into three categories -
- Blocking or Rejecting
	- requests are rejected if it has exceeded the threshold
	- eg - limiting the requests on API level
- Throttling or Slowing
	- requests are not rejected but are slowed down
	- they are kept in buffer to be processed later, here rate limiter act as a buffer (message queue)
- Shaping
	- requests that has exceeded the threshold are processed and not rejected but they are assigned lower priority
- Ignoring
	- requests are ignored if it has exceeded the threshold but clients are not informed


### Common Rate Limiting Algorithms

#### Fixed Window Counter
- divides the timeline into fixed size time windows and assign a counter for each window
- whenever request comes it increments the counter by some value or by 1
- once the counter reaches the threshold in a time window, all the subsequent requests are dropped until a new time window starts
- eg - let's say the system allows only 5 requests per minute, if the system receives more than 5 requests in 1 min time window then all the subsequent requests are dropped
- Cons - burst of traffic at the end or at the beginning of the time window could allow excess requests over the threshold to go through
- Pros -
	- Easy to understand
	- Memory efficient

#### Sliding Window Log
- resolves the issue introduced by Fixed window counter
- it keeps track of the timestamp of requests in a log, the log is usually kept in a cache like sorted set in Redis
- when a request arrives, it's timestamp is added to the log and following things are checked
	-  all the outdated request timestamps are removed from the log. Outdated timestamps are those older than the start of the current time window
	- if the request is within the current time window and it is within the threshold then it is allowed
	- if the request is within the current time window and it has exceeded the threshold then it is rejected even though it's timestamp is added to the log
- Pros -
	- in any rolling window, requests will not exceed the threshold
- Cons -
	- Not memory efficient as timestamp might remain in the log even though the request is rejected

#### Sliding Window Counter
- hybrid approach that combines the Fixed Window Counter and the Sliding Window log
- need to know the no. of requests in a sliding or rolling window
- Instead of using the log to keep track of the timestamp of requests, it uses two approach
	- Weighted counter for the previous time window
		- when a new request arrives, the counter is adjusted based on the weight and the request is allowed if its within the limit
		- no. of requests in the rolling window = requests in the current window + requests in the previous window * overlap percentage of the rolling and previous window
	- uses counter for each time slot within the current window
- Pros -
	- Memory efficient
	- Smoothes out spikes in traffic as the rate is based on the average rate of the previous window
- Cons -
	- could still allow bursts of traffic as it assumes the requests in the previous window are evenly distributed

#### Token Bucket
- holds tokens in a bucket
- tokens represent the no. of allowed requests
- tokens are added into the bucket at a fixed rate (add 2 tokens every 1 min)
- when a request comes, if there are enough tokens in the bucket, it consumes the token and the request goes through else if there are not enough tokens the request is dropped
- used by - Amazon and Stripe
- this algorithm requires two parameters 
	- bucket size - max tokens allowed in a bucket
	- refill rate - rate at which the tokens are added into a bucket
- Pros -
	- Simple and easy to understand and implement
	- Memory efficient
	- used by multiple tech companies (Amazon, Stripe)
	- allows burst of traffic as long as there are enough tokens

#### Leaky Bucket
- requests are processed at a regular interval
- it is implemented using FIFO queue
- when a request comes the system first check if the queue is full or not, if the queue is full then the request is dropped else the request is added to the queue
- requests are then pulled from the queue and processed at a regular interval
- used by - Shopify
- this algorithm requires two parameters 
	- bucket size - the size of the queue
	- outflow rate - defines how many requests can be processed at a fixed rate i.e. no. of requests per sec
- Pros -
	- Memory efficient
	- Good for application that requires stable outflow rate