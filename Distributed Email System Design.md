
### APIs Design
- send email
	- POST - /v1/message
	- Request Body - (from, to, cc, bcc, subject, body, attachments)
- get all folders
	- GET - /v1/folders
	- e.gs - inbox, sent, draft, starred, important, spam, trash
- get all emails inside a folder
	- GET - /v1/folders/{folderId}/messages
- get a particular email info
	- GET - /v1/messages/{messageId}
	- Response Body
		- from - (name, email) 
		- to - list of (name, email)
		- cc, bcc
		- subject
		- body
		- attachments