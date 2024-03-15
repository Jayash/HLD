# Transactions

Implementing fault-tolerant mechanisms is a lot of work.

## The slippery concept of a transaction

Transactions have been the mechanism of choice for simplifying these issues. Conceptually, all the reads and writes in a transaction are executed as one operation: either the entire transaction succeeds (commit) or it fails (abort, rollback).

The application is free to ignore certain potential error scenarios and concurrency issues (safety guarantees).

### ACID
- Atomicity. Is not about concurrency. It is what happens if a client wants to make several writes, but a fault occurs after some of the writes have been processed. Abortability would have been a better term than atomicity.
- Consistency. Invariants on your data must always be true. The idea of consistency depends on the application's notion of invariants. Atomicity, isolation, and durability are properties of the database, whereas consistency (in an ACID sense) is a property of the application.
- Isolation. Concurrently executing transactions are isolated from each other. It's also called serializability, each transaction can pretend that it is the only transaction running on the entire database, and the result is the same as if they had run serially (one after the other).
- Durability. Once a transaction has committed successfully, any data it has written will not be forgotten, even if there is a hardware fault or the database crashes. In a single-node database this means the data has been written to nonvolatile storage. In a replicated database it means the data has been successfully copied to some number of nodes.

Atomicity can be implemented using a log for crash recovery, and isolation can be implemented using a lock on each object, allowing only one thread to access an object at any one time.

A transaction is a mechanism for grouping multiple operations on multiple objects into one unit of execution.

### Handling errors and aborts
A key feature of a transaction is that it can be aborted and safely retried if an error occurred.

In datastores with leaderless replication is the application's responsibility to recover from errors.

The whole point of aborts is to enable safe retries.

## Weak isolation levels
Concurrency issues (race conditions) come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.

Databases have long tried to hide concurrency issues by providing transaction isolation.

In practice, is not that simple. Serializable isolation has a performance cost. It's common for systems to use weaker levels of isolation, which protect against some concurrency issues, but not all.

Weak isolation levels used in practice:

### Read committed
It makes two guarantees:

- When reading from the database, you will only see data that has been committed (no dirty reads). Writes by a transaction only become visible to others when that transaction commits.
- When writing to the database, you will only overwrite data that has been committed (no dirty writes). Dirty writes are prevented usually by delaying the second write until the first write's transaction has committed or aborted.

Most databases prevent dirty writes by using row-level locks that hold the lock until the transaction is committed or aborted. Only one transaction can hold the lock for any given object.

On dirty reads, requiring read locks does not work well in practice as one long-running write transaction can force many read-only transactions to wait. For every object that is written, the database remembers both the old committed value and the new value set by the transaction that currently holds the write lock. While the transaction is ongoing, any other transactions that read the object are simply given the old value.

### Snapshot isolation and repeatable read
There are still plenty of ways in which you can have concurrency bugs when using this isolation level.

Nonrepeatable read or read skew, when you read at the same time you committed a change you may see temporal and inconsistent results.

There are some situations that cannot tolerate such temporal inconsistencies:

- Backups. During the time that the backup process is running, writes will continue to be made to the database. If you need to restore from such a backup, inconsistencies can become permanent.
- Analytic queries and integrity checks. You may get nonsensical results if they observe parts of the database at different points in time.

Snapshot isolation is the most common solution. Each transaction reads from a consistent snapshot of the database.

The implementation of snapshots typically use write locks to prevent dirty writes.

The database must potentially keep several different committed versions of an object (multi-version concurrency control or MVCC).

Read committed uses a separate snapshot for each query, while snapshot isolation uses the same snapshot for an entire transaction.

How do indexes work in a multi-version database? One option is to have the index simply point to all versions of an object and require an index query to filter out any object versions that are not visible to the current transaction.

Snapshot isolation is called serializable in Oracle, and repeatable read in PostgreSQL and MySQL.

### Preventing lost updates
This might happen if an application reads some value from the database, modifies it, and writes it back. If two transactions do this concurrently, one of the modifications can be lost (later write clobbers the earlier write).

### Atomic write operations
A solution for this it to avoid the need to implement read-modify-write cycles and provide atomic operations such us

`UPDATE counters SET value = value + 1 WHERE key = 'foo';`

MongoDB provides atomic operations for making local modifications, and Redis provides atomic operations for modifying data structures.

### Explicit locking
The application explicitly lock objects that are going to be updated.

### Automatically detecting lost updates
Allow them to execute in parallel, if the transaction manager detects a lost update, abort the transaction and force it to retry its read-modify-write cycle.

MySQL/InnoDB's repeatable read does not detect lost updates.

### Compare-and-set
If the current value does not match with what you previously read, the update has no effect.

`UPDATE wiki_pages SET content = 'new content'
  WHERE id = 1234 AND content = 'old content';`

### Conflict resolution and replication
With multi-leader or leaderless replication, compare-and-set do not apply.

A common approach in replicated databases is to allow concurrent writes to create several conflicting versions of a value (also know as siblings), and to use application code or special data structures to resolve and merge these versions after the fact.

### Write skew and phantoms
Imagine Alice and Bob are two on-call doctors for a particular shift. Imagine both the request to leave because they are feeling unwell. Unfortunately they happen to click the button to go off call at approximately the same time.

```
ALICE                                   BOB

┌─ BEGIN TRANSACTION                    ┌─ BEGIN TRANSACTION
│                                       │
├─ currently_on_call = (                ├─ currently_on_call = (
│   select count(*) from doctors        │    select count(*) from doctors
│   where on_call = true                │    where on_call = true
│   and shift_id = 1234                 │    and shift_id = 1234
│  )                                    │  )
│  // now currently_on_call = 2         │  // now currently_on_call = 2
│                                       │
├─ if (currently_on_call  2) {          │
│    update doctors                     │
│    set on_call = false                │
│    where name = 'Alice'               │
│    and shift_id = 1234                ├─ if (currently_on_call >= 2) {
│  }                                    │    update doctors
│                                       │    set on_call = false
└─ COMMIT TRANSACTION                   │    where name = 'Bob'  
                                        │    and shift_id = 1234
                                        │  }
                                        │
                                        └─ COMMIT TRANSACTION
```
