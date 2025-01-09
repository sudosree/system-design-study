## Why do we need locking ?
- In an online reservation system like flight booking or ticket booking system when a user tries to book a seat the system needs to ensure that during that time no other user can book the same seat
- The above can be achieved by acquiring a lock on a seat for a short period of time
- Distributed locks are useful when you need to lock something through different systems for a reasonable period of time
- It's not similar to mutex in multi threaded env
- When system1 or client1 acquires a lock to reserve a seat then in that time client2 won't be able to acquire the lock on the same seat for client2 that seat will be unavailable, if client1 is able to book the seat before the lock expires then that seat becomes unavailable for other clients, else if not and if the TTL of the lock expires then the seat becomes available for other clients and client2 can acquire lock on it
- It can be achieved using key value store like Redis
- we can store a lock in a key value store and use the atomicity in Redis to ensure that only one process at a time can acquire the lock
- When we try to acquire a lock we use the atomic increment INCR, if the response is 1 then we acquire the lock, if the response is > 1 then some other process has the lock and we need to wait and when we are done with the lock we can DEL the key so that other process can acquire the lock
- Let' say we have a key ticket-123 and we want to lock it we can set the value of the key to locked and when we are done with the processing of the key we unlock it
- we should set a TTL for a lock so that it expires after a certain period of time, it ensures that a lock doesn't get stuck in a locked state when a system crashes or is killed
- Distributed locks can be applied on a single resource or on a group of resource
- Use cases for distributed locking -
- Reservation system
- Ecommerce checkout system
- Ride sharing matchmaking
- There are Different ways to implement distributed lock - one implementation uses Redis called Redlock. 
- Redlock uses multiple redis instances to ensure that a lock is acquired and released in a safe manner


## Pessimistic Locking
- Acquire a lock on a resource, thread that acquires the lock proceeds and other threads needs to wait
- wraps the critical section around a lock
- If there are n threads only one thread at a time will acquire the thread and executes the critical section while other thread needs to wait, once the execution is done it releases the lock
- eg - reservation system
- acquire a lock on a seat for a short period of time (reserves the seat) and during that time no other users can book the seat for them the seat is not available
- If you are able to book the seat during the given period of time then the booking is successful and the lock will be released else if you are unable to book the seat during the given period of time then the lock will be released once the timer expires and other users should see the seat as available and can book it
- Very slow performance, reduces the throughput of the system


## Optimistic Locking
- When two threads tries to do the same thing at a same time, then one will succeed and the other will fail
- Allowing multiple transactions to do the same thing (update the same resource) at the same time
- do not use locking on resource
- can be implemented using compare and swap
- Advantages -
- better throughput of the system
- low locking overhead
- we get to pick how do we resolve the conflicts (when one thread fails we can either retry or ignore or throw exception)
- useful when  contention is low (rare conflicts)
- Disadvantages -
- Non trivial and verbose implementation
- not meant for all usecases
- conflicts are high or contention is high
- performance reduces due to repeatable retries