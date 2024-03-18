# Consistency and consensus

The simplest way of handling faults is to simply let the entire service fail. We need to find ways of tolerating faults.

## Consistency guarantees

Write requests arrive on different nodes at different times.

Most replicated databases provide at least eventual consistency. The inconsistency is temporary, and eventually resolves itself (convergence).

With weak guarantees, you need to be constantly aware of its limitations. Systems with stronger guarantees may have worse performance or be less fault-tolerant than systems with weaker guarantees.

## Linearizability

Make a system appear as if there were only one copy of the data, and all operaitons on it are atomic.

read(x) => v Read from register x, database returns value v.
write(x,v) => r r could be ok or error.
