### Functional Requirements
- generate short and unique alias - https://tinyurl.com/abc123e5
- length of the short alias ???
- short alias should contain alphanumeric characters ([0-9][a-z][A-Z])
- custom short alias ???
- short alias should expire after a default expiration time
- custom expiration time ???

### Non-Functional Requirements
- Highly available
- Highly scalable
- Fault tolerant (No SPOF)

### Back of the envelope calculation
- 100 million urls should be generated per day
- Urls generation requests per month = 30 * 100 million = 3 billion
- Read: Write ratio = 10:1
- Urls redirection requests per day = 10 * 100 million = 1 billion
- Urls redirection requests per month = 30 billion
- QPS (Queries per second) for URL generation = 100 million / 24 / 3600 = 1160
- QPS (Queries per second) for URL redirection = 10 * 1160 = 11600
- URL generation requests for 10 years = 3 billion * 12 * 10 = 360 billion
- Length of each URL = 100 bytes
- Storage required for 10 years = 100 bytes * 360 billion = 36 TB

### APIs
- shortenUrl(longUrl, customAlias) -> shortUrl
- redirectUrl(shortUrl) -> longUrl
- expireUrl(shortUrl, expireTime)