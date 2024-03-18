# Consistency and consensus

The simplest way of handling faults is to simply let the entire service fail. We need to find ways of tolerating faults.

## Consistency guarantees

Write requests arrive on different nodes at different times.

Most replicated databases provide at least eventual consistency. The inconsistency is temporary, and eventually resolves itself (convergence).

With weak guarantees, you need to be constantly aware of its limitations. Systems with stronger guarantees may have worse performance or be less fault-tolerant than systems with weaker guarantees.

## Linearizability

Make a system appear as if there were only one copy of the data, and all operaitons on it are atomic.

- read(x) => v Read from register x, database returns value v.
- write(x,v) => r r could be ok or error.

If one client read returns the new value, all subsequent reads must also return the new value.

- cas(x_old, v_old, v_new) => r an atomic compare-and-set operation. If the value of the register x equals v_old, it is atomically set to v_new. If x != v_old the registers is unchanged and it returns an error.

Serializability: Transactions behave the same as if they had executed some serial order.

Linearizability: Recency guarantee on reads and writes of a register (individual object).

### Locking and leader election
To ensure that there is indeed only one leader, a lock is used. It must be linearizable: all nodes must agree which nodes owns the lock; otherwise is useless.

Apache ZooKeepr and etcd are often used for distributed locks and leader election.

### Constraints and uniqueness guarantees
Unique constraints, like a username or an email address require a situation similiar to a lock.

A hard uniqueness constraint in relational databases requires linearizability.

### Implementing linearizable systems

The simplest approach would be to have a single copy of the data, but this would not be able to tolerate faults.

- Single-leader repolication is potentially linearizable.
- Consensus algorithms is linearizable.
- Multi-leader replication is not linearizable.
- Leaderless replication is probably not linearizable.

Multi-leader replication is often a good choice for multi-datacenter replication. On a network interruption betwen data-centers will force a choice between linearizability and availability.

With multi-leader configuraiton, each data center can operate normally with interruptions.

- With single-leader replication, the leader must be in one of the datacenters. If the application requires linearizable reads and writes, the network interruption causes the application to become unavailable.

- If your applicaiton requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, the some replicas cannot process request while they are disconnected (unavailable).

If your application does not require, then it can be written in a way tha each replica can process requests independently, even if it is disconnected from other replicas (peg: multi-leader), becoming available.

If an application does not require linearizability it can be more tolerant of network problems.

### The unhelpful CAP theorem
CAP is sometimes presented as Consistency, Availability, Partition tolerance: pick 2 out of 3. Or being said in another way either Consistency or Available when Partitioned.

CAP only considers one consistency model (linearizability) and one kind of fault (network partitions, or nodes that are alive but disconnected from each other). It doesn't say anything about network delays, dead nodes, or other trade-offs. CAP has been historically influential, but nowadays has little practical value for designing systems.

--- 

The main reason for dropping linearizability is performance, not fault tolerance. Linearizabilit is slow and this is true all the time, not on only during a network fault.

## Ordering guarantees
