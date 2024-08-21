
### Functional Requirements
- users should be able to search or view the list of hotels by -
	- place
	- check in and check out date
- users should be able to view the availability of rooms by -
	- check in and check out date
	- no. of guests (adult, children)
	- no. of rooms
- users should be able to filter their hotel search by
	- ratings
	- reviews
- users should be able to make reservations or booking
- users should be able to cancel any reservation
- How the payment will be made ?
	- Full payment
	- Partial payment
	- Payment at the property
- Scale of the system ??? 5000 hotels and 1 million rooms in total

### Non Functional Requirements
- High Concurrency
	- there will be multiple booking requests at the same time at the same room
- Availability can be a trade off for consistency
- Low latency

### Back of the envelope estimation
- No. of reservations per month = 30 Million
- No. of page views per month (view list of hotels, view hotel and room details page) = 3000 Million
- Read : Write ratio = 100 : 1
- no. of read requests per second (QPS)
	- Per day = 3000 M / 30 = 300 M
	- Per sec = 300 M / 100000 = 3000 TPS
- no. of write requests per second (QPS)
	- per sec = 3000/100 = 30 TPS
- no. of hotels = 10 Million
- avg no. of rooms per hotel = 30
- no. of room types = 20
- total no. of rooms = 300 Million


### High Level Design

#### API Design
- ##### Hotel Related APIs
	- Get list of hotels by place - getHotels(place) - GET /v1/hotels?place={place}
	- Get the details of a hotel - getHotel(hotelid) - GET /v1/hotels/{hotelid}
	- Add a hotel to the system (only hotel admins can do it) - addHotel(hotelName, place) - POST /v1/hotels
	- Update a hotel (only hotel admins can do it) - updateHotel(hotelId) - PUT /v1/hotels/{hotelid}
	- Delete a hotel (only hotel admins can do it) - deleteHotel(hotelId) - DELETE /v1/hotels/{hotelid}
	  
- ##### Room Related APIs
	- Get list of rooms in a hotel - getRooms(hotelId) - GET /v1/hotels/{hotelId}/rooms
	- Get the availability of rooms in a hotel - getRooms(hotelId, checkInDate, checkOutDate, # guests, # rooms) - GET /v1/hotels/{hotelId}/rooms?checkInDate={checkInDate}&checkOutDate={checkOutDate}&guests={guests}&rooms={rooms}
	- Get the details of a room - getRoom(hotelId, roomId) - GET /v1/hotels{hotelId}/rooms/{roomId}
	- Add a room to the hotel (only hotel admins can do it) - addRoom(hotelId, roomName) - POST /v1/hotels/{hotelid}/rooms
	- Update a room (only hotel admins can do it) - updateRoom(hotelId, roomId) - PUT /v1/hotels/{hotelid}/rooms/{roomId}
	- Delete a room (only hotel admins can do it) - deleteRoom(hotelId, roomId) - DELETE /v1/hotels/{hotelid}/rooms/{roomId}
	  
- ##### Reservations Related APIs
	- Get reservation history of a user - GET /v1/reservations
	- Get the detail of a reservation - GET /v1/reservations/{reservationId}
	- Make a reservation - POST /v1/reservations - makeReservation(hotelId, roomTypeId, checkInDate, checkOutDate, noOfRooms, noOfGuests)
	- Cancel a reservation - DELETE /v1/reservations/{reservationId}

#### Database Design
- Relational database should be used (Why ????)
	- Need high concurrency and SQL database supports ACID guarantees
	- The data is structured and there are relationships between entities
	  
- ##### Data Model
hotel 

| hotel_id | name | address | email | phone_number | location |
| -------- | ---- | ------- | ----- | ------------ | -------- |
|          |      |         |       |              |          |
room

| room_id | hotel_id | type | name | number | status | floor |
| ------- | -------- | ---- | ---- | ------ | ------ | ----- |
|         |          |      |      |        |        |       |
reservation

| reservation_id | hotel_id | room_type_id | checkin_date | checkout_date | guest_id |
| -------------- | -------- | ------------ | ------------ | ------------- | -------- |
|                |          |              |              |               |          |
inventory

| hotel_id | room_type_id | date | total_inventory | total_reserved |
| -------- | ------------ | ---- | --------------- | -------------- |
|          |              |      |                 |                |
guest

| guest_id | name | email | phone_number |
| -------- | ---- | ----- | ------------ |
|          |      |       |              |

#### Components
- Hotel Service
	- provides information on hotels and rooms
- Reservation Service
	- provides information on reservation history
	- receives request to make a reservation and cancel a reservation
- Inventory Service
	- provides information on availability of rooms
- Hotel Management Service
	- receives request from admins to add hotel and rooms to the system


### Design Deep Dive

#### Reservation Flow
- user makes a reservation request to Reservation Service
- system first checks the availability of rooms from the Inventory service
- If the rooms are available then user can proceed with the reservation
- How to check the availability of rooms ?
	- total_reserved + no_of_rooms_to_reserve <= total_inventory
- How data will be stored in the Inventory database ?
	- data will be stored on the basis of date
	- need to pre-populate the data for 2 years
	- cron job will run daily and populate the data for the current date
- Storage required for the Inventory database
	- 10 M hotels * 20 room types * 365 * 2 = 146000 M = 146 B = 146 GB
	- not much data -> can be stored in a single database
	- to achieve high availability need to replicate it
- How past data will be handled ?
	- data for past dates are not required
	- cron job will run daily and will remove the data for the past dates
- Query to get the no. of rooms from the Inventory database based on the checkin and checkout date
	- select date, total_inventory, total_reserved from inventory where hotel_id = {hotel_id} and room_type_id = {room_type_id} and date between {checkindate} and {checkoutdate}
	- for each record check the availability of rooms by
		- total_reserved + no_of_rooms_to_reserve <= total_inventory
- Storage required for the Reservation database
	- No. of reservations per month = 30 Million = 30 MB (can be stored in single database)
	- How will you handle scale if the reservation increases ?
		- shard the database based on hotel_id
	- replicate the database to achieve high availability
- How will you handle multiple booking requests from different users for the same hotel and room type at the same time ?
	- pessimistic locking
		- prevents users from concurrently updating the same record by applying locks
		- Pros
			- prevents application from updating the data that is being changed or has been changed
			- useful when data contention is high
		- Cons
			- introduces deadlock as multiple resources are blocked
			- not scalable because if a transaction has been locked for long time it degrades the database performance
	- optimistic locking
		- allows users from concurrently updating the same record 
		- doesn't use locks
		- Pros
			- use version vector to implement it
			- prevents application from updating the stale data
			- no deadlock
			- useful when data contention is low
		- Cons
			- not useful when data contention is high
	- database constraint
		- total_inventory - total_reserved <= 0

#### Caching

