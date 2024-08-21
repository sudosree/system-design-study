- Multiple leaders accepts the write requests and multiple followers accepts the read requests
- Pros - No single point of failure for write requests
- Cons - write conflicts as write requests are handled by multiple leaders
- Use cases - MultiData Center, Google docs
- Each multidata center has one leader replica and multiple follower replicas
- Across datacenters asynchronous replication is used between leaders
- Within datacenter single leader replication is used

## Handle write conflicts
- Last Write Wins -> Assign a timestamp to each write request, the request with the latest timestamp should be considered
  Cons - loss of data
- Store multiple conflict version of the same data in every node and returns them to the client during the read operation. Client will resolve it using merging techniques like CRDTs etc.

## Detect write conflicts or concurrent writes
- Write conflicts can be detected using version vectors or vector clock
- Multiple version of the same data are returned to the client during read operation if the writes are not resolved by the system or they are concurrent writes 
- Clients merge those concurrent writes using merging techniques like CRDTs etc