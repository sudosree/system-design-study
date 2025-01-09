
### Relational Database 
Main features are -
- SQL Joins
- Indexing
- Transactions
- ACID properties
- egs - Postgres, MySQL

### Non Relational Database
can be categorized into
- key value stores
	- data is stored as key value pair
	- fast access
	- simple model
	- provides high scalability, availability, fault tolerant, high speed read
	- eg - DynamoDB, Redis Cache
- document stores
	- Flexible data model 
	- schema less
	- useful when your data model is continuously evolving
	- eg - MongoDB, Couchbase
- column family stores
	- data is stored in columns rather than rows
	- provides high scalability
	- provides high performance write
	- useful when need to store and query large amount of data
	- eg - Cassandra, HBase
- graph databases
	- useful when there is relationship like social network
	- efficient retrieval
	- eg - Neo4j, Amazon Neptune

Features of NoSQL
- Different data models
- Consistency models - from strong to eventual consistency
- Indexing
- Scalability


### Blob Storage
- stores large data like images, videos, files
- Features -
- durable - data once stored are not lost
- scalable
- cost effective
- security
- chunking
- eg - Amazon S3, Google Cloud Storage

### Search Optimized Database
- efficient for full text search (it is the ability to search through a large amount of text data and find the relevant results)
- makes search queries fast and efficient
- uses technique like tokenization, indexing, stemming and inverted indexes
- Inverted indexes - is a data structure that maps from words to documents that contains them
- Tokenization - breaks down a piece of text into individual words. This allows us to map from words to documents
- Stemming - process of reducing words to their root form. It allows us to search different forms of the same word. 
- supports Fuzzy Search - is the ability to find results that are similar to a given search term
- can scale well
- eg - ElasticSearch

### 