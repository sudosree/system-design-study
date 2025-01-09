- It is a transaction that spans across multiple systems and needs to be performed atomically
- To make a transaction atomic, if a transaction fails on some nodes but succeeds on others then abort or rollback the transaction, if a transaction succeeds on all the nodes then commit the transaction
- Atomic transaction is a transaction where you get all the nodes to agree on the outcome of the transaction - either they all abort the transaction if anything goes wrong or commit the transaction if nothing goes wrong

### Two Phase Commit
- an algorithm to solve the atomic transaction commit across multiple nodes
- consists of two phase - prepare or reserve and commit or assign
- In airline reservation system to achieve atomic transaction guarantee during seat booking, following things happen -
- Seats are first reserved i.e. they are not booked but they are made unavailable for other users to book it. This is done for some short period of time using a timer. (reserve phase - 1st phase)
- When seats are reserved successfully then it is booked (commit phase - 2nd phase) and tickets are sent to users
- If either of them fails the entire transaction is aborted or rollback else if both of them succeeds the transaction is committed.
- Performance of 2PC is very slow as it guarantees strong consistency 