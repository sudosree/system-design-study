### Functional Requirements
- Users should be able to upload/view/share/like videos
- Users should be able to add/view comments on videos
- Video streaming should be smooth
- Clients supported - Mobile apps, web browser, smart TV
- Should support all video resolutions format
- Max file size - 1GB

### Non Functional Requirements
- Highly available, scalable and reliable (video once uploaded should not be lost)
- Low latency
- Shouldn't be any lag during video streaming


### Back of the envelope estimation
- Daily active users (DAU) = 5 millions
- users watch 5 videos per day on avg
- total videos watch per day = 25 millions
- 10 % users upload 1 video per day = 10 % * 5M = 500,000 users * 1 video =  500,000  videos uploaded per day
- avg video file size = 300 MB
- upload:view ratio = 1:50
- videos upload per sec = 500,000/10^5 = 5
- videos view per sec = 25 millions/10^5 = 250
- Storage reqd per day = 5 M * 10% * 300 MB = 5 * 10^5 * 300 * 10^6 = 150 * 10^12 = 150 TB
- CDN Cost
- avg cost per GB = 0.02$
- total cost = 0.02$ * 0.3GB * 5 * 5 M = 150,000$ per day


### High Level Design

