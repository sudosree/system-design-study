
## Transaction
- Transactions are set of operations that are grouped together as a single unit
- 

### Atomicity
- guarantees that either the entire transaction succeeds or the entire transaction fails
- if all the operations within a transaction are successfully completed then the entire transaction is considered as successful
- and if there is any failure in a transaction then the entire transaction fails or aborts or rollback
- e.g - Banking system

### Consistency
- guarantees that the database remains in a consistent state before and after a transaction
- all the data integrity constraints such as unique, foreign key and check constraints are satisfied before and after a transaction
- if a transaction violates any rules then it will not be committed and the database will revert to its previous state

### Isolation
- guarantees that transactions do not interfere with other transactions
- a transaction do not see the intermediate state of other transactions until they are committed
- e.g - if there are multiple write operations within a transaction then other transaction either sees all those write or none of those writes but it shouldn't see an inconsistent halfway state
- prevents data inconsistencies
- different levels of isolation
	- Read uncommitted 
		- Transaction can see the uncommitted changes from other transactions
	- Read committed
		- Transaction can only see the committed changes from other transactions
	- Repeatable read
		- ensures that when a transaction reads a row, it will read the same row consistently throughout its execution
	- Serializable
		- highest form of isolation, guarantees that the transactions are always executed serially

### Durability
- guarantees that transaction remains durable once it is committed even after system failure
- data is durable and it is not lost once it is committed even after system failure
- ways to achieve durability
	- write data to disk
	- replicate data across multiple nodes
	- take backup of data as well


### Weak Isolation Levels
- #### Read Committed Isolation
	- most basic level of transaction isolation
	- makes two guarantees -
		- no dirty reads
			- when reading data from the database, it should see the data that has been committed
		- no dirty writes
			- when writing data to the database, it should overwrite the data that has been committed
	- dirty reads - say there are two transactions T1 and T2, T1 has written some data to the database but the transaction is not committed or aborted and another transaction T2 sees the uncommitted data
	- dirty writes - say there are two transactions T1 and T2, and both tries to update the same record concurrently, we don't know in which order the writes will happen. If the earlier write is part of a transaction that has not been committed that means the later writes on the uncommitted data
	- Reasons to prevent dirty reads -
		- If a transaction has several write operations, if dirty read is allowed it means the other transaction may see some updates or writes but not others, it causes confusion to users and causes the other transaction to take incorrect decision
		- If a transaction is aborted then any writes it has made needs to be rolled back otherwise it sees the data that has not been committed
	- How do database implement no dirty writes ? 
		- by acquiring lock on row level object, T1 and T2 wants to modify the same object, T1 acquires the write lock first on the row and modifies the object, T2 needs to wait for the lock to be released by T1 and then it can modifies it
	- How do database implement no dirty reads ? - 
		- we can implement it using locks but it will take performance hit
		- another approach - keeps track of the old committed value and the new value holds by the transaction that acquires the write lock. Before this transaction is committed, for any other transaction old committed value is returned once this transaction is committed then new value is returned.

### Serializable Isolation
- means the database ensures that when transactions have committed, the result is the same as if they had run serially even though in reality they may have run concurrently