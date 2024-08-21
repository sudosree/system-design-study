
### Functional Requirements
- users should be able to upload and download files
- users should be able to share files with other users
	- users should be notified when any files or folders are shared with them
- Type of files supported - text files, docs, sheets (xlsx), media files (photos, videos)
- File size limit - upto 10 GB
- Files should be in sync across multiple devices or clients i.e. when a user uploads a file from their mobile device it should be available from other devices as well
- users should be able to view the revision history of a file
- users should be able to edit and delete files online and offline
	- when they come online all their changes will be synced to the server
- multiple users should be able to edit files - optional (Google Doc collaboration)

### Non Functional Requirements
- Scalability - should support high volume of data
- Reliability and durability - once a file is uploaded it should be available in the storage, there is no loss of data, files are replicated across different geographic locations
- High Availability - files can be accessed from anywhere at anytime
- Files synchronisation should be fast (low latency) and should be consistent
- 
- Read and Write ratio 1:1


### Back of the envelope estimation
- Total users - 500 M
- DAU - 100 M
- on avg each user uploads 2 files per day
- avg file size - 100 KB
- each user gets 15GB free space
- Avg file uploads per day = 200 M
- storage reqd for all files per day = 200 M * 100 KB = 50 TB
- storage reqd for all users = 500 M * 15 GB = 7500 PB


### API Design
- Upload a file
	- https://drive.googleapis.com/files/upload
		- Params -
			- uploadType
			- file
- Download a file
	- https://drive.googleapis.com/files/download
		- Params - path
- Revision history of files
	- https://drive.googleapis.com/files/revisions
		- Params -
			- path
			- limit

### High Level Design
- Simple Upload Flow Design
	- user will send a request to upload servers
	- upload servers will save the file in some storage system database and save the file metadata information in database (SQL or NoSQL)
	- Storage system database has total capacity of 10 TB (not enough space as eventually you will run out of space)
	- 1st solution - shard the database based on the user id (cons - not enough space as eventually you will run out of space)
	- 2nd solution - Object storage like Amazon S3

- Block servers will be responsible for uploading and downloading files to/from the cloud storage
- Metadata information like files, users, file revision history will be stored in metadata database
- Notification service / Synchronization service -> will be responsible for sending out the notification to clients whenever a file is modified or deleted by some other clients


1. Why Amazon S3 ?
	- provides scalability, durability, availability, security
	- supports same region and cross region replication

### Database Design
- SQL or NoSQL for metadata database
	- SQL because it gurantees ACID properties
	- Files should be in sync or consistent across multiple clients or devices, it should be highly consistent
	- If a user modifies a file from mobile device the modified version should be visible across all the other devices
#### Metadata Database
- User Table

| user_id | user_name | user_email | created_at |
| ------- | --------- | ---------- | ---------- |
|         |           |            |            |
- Device Table

| device_id | user_id | last_logged_in_at |
| --------- | ------- | ----------------- |
|           |         |                   |
- File Table

| file_id | file_name | file_type | file_size | uploaded_by | last_uploaded_at | last_modified_at | object_storage_path | is_shared | shared_with | modified_by | file_upload_status |
| ------- | --------- | --------- | --------- | ----------- | ---------------- | ---------------- | ------------------- | --------- | ----------- | ----------- | ------------------ |
|         |           |           |           |             |                  |                  |                     |           |             |             |                    |
- File Revision History Table

| id  | file_id | file_version_id | last_modified_at |
| --- | ------- | --------------- | ---------------- |
|     |         |                 |                  |
- Block Table
	- a file has two versions and each version has 4 blocks so total there will be 8 entries in the block table

| block_id(unique hash value of a block) | file_version_id | block_order |
| -------------------------------------- | --------------- | ----------- |
|                                        |                 |             |

### Design deep dive
- Block Servers
	- split the file into multiple smaller blocks
	- compress each block using compression algorithm (reduces the size of blocks)
	- encrypts the blocks and then upload the blocks to the cloud storage
	- whenever a file is modified only modified blocks are uploaded to the cloud storage using a sync algorithm
- Metadata Database (refer database design)
- Notification service / Synchronization service Flow
	- Client A (mobile device) updates the file then metadata information of the file will be updated in the Metadata DB and also the updated blocks will be stored in the cloud storage or object storage
	- Client B and Client C should also receive this update information and should see the updated version of the file
	- If both the clients B and C are online they will establish a long poll connection with the Notification service
	- If a NS receives any file update change it will send the information to both the clients
	- Clients will close the long poll connection and will connect to the API servers and Block servers to get the metadata info and download the file from the cloud storage
	- When clients are offline then update info will be sent to the message queues from the NS and when they come online they will get the data from the message queues (clients needs to subscribe to the message queues)

