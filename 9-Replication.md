# Replication

Quality drivers:
- Increase relibaility: switch copies if one does not live up to specs
- Performance: spread request between differetn replicated parts to keep the laod balanced, and taking proximity into account.

> Partitionig or Sharding means splittng the data into smaller subsets and distribute them over multiple nodes

> Replication is distribting multiple copies of the same data set over distinct nodes

Often combined in practice, multiple repllicas per partition.

## Consistency

A modification on replica A, needs to propagated to other replicas. Two broad models:

1. Data-centric models
2. Client-centric models


Two operations are conflicting if: the outcome of executing them as two atmoic actions may potentially differ per ordering of actions.

Conflicing operations come in two flavours:

1. Read-write conflicts: a read and write operation act concurrentely. Order here matters -> hence conflicting
2. Write-write conflicts: two concurrent write operations may leave the store in differetn states. Ordering also matters.

OS handles this by using locks/mutex, semaphores, atomic operations and transactions

Notions:
- $W_i(x)a$: Process $i$ writes value of $a$ to $x$
- $R_i(x)a$: Process $i$ reads value $a$ from $x$
- All data items have initial value of $\textsc{nil}$

### Data centric model
The data store specifies precisely what the result of raed and write operations are in the presence of concurrency.

A contract between data stores and processes: if processe obey some rules, the data store works as promised.

> Sequential consistency: The result of any execution is the same as if all read  and write operations done by processes on the data store happened in some sequential order, and the operations of each invididueal process appear in this sequence in the order specified by its program.

So: all concurrent operations are equivalent to some sequental ordering. And in this ordering, the operations done by some client, must appeart in this ordering the same as that they were actually performed by the clients program

Middleware propagates writes to other stores. Must do this in a way that maintains sequential consistency.


> Linearizabiilty: each operation should appear to take effect instantaneously at t some moment between its start and completion

> Causal consistency Writes that are potentially causally related must be seen by all processes in the same order. Concurrent writes may be seen in a different order by different machines.

If process A reads value x and write value y, it might be causally related ,because the write of y mnight not have occurred if the value x would have been different. Hence, assume it is.

This is weaker than sequential consistency and is determined by means of time stamps 


> Eventual consistency: when in lakck of updates of a certain moment, the updates of data stores propagates in such a way that all stores are consistent.

Bare minimum, weaker the causal consistency.

The rules for processes: issue operations in a well-defined order, and wait on acknolowdgements when dependenceis exist. So no concurrent writes write/reads in a single process.

## Client centric model

This type of consistency provdies guarantees for a single client concerning the consistency of accesse to a data store by that client.

### Monotonic read consistency

> If you read a value, you will never see an older value in the future

### Monotonic write consistency

> Writes from a client are executed in the order that the client issued

If you update something on location 2, then all updates you did on location 1 will first be done on l2.

### Read your writes consistency

> The effect of a write operation on data item x is always visible in subsequent read operations by the same process.

### Writes follow reads consistency

> If a client reads a value, any subsequent write will be done on that value or a newer one.

# Content replication

A process is cpaable of hosting a replica of an object or data
- permanent replicas: process/machine always havinga replica
deafult replicas that are always there
- server-initated replica: process that can dynamically host a replica on request of another server inthe data store
useful on a burst of incoming traffic, set up some temporary replicas
- client-inititaed replica: process that can dyncamilly host a replica on reqeust of cleint (clien caceh)
local cache on the client, server _may_ assist in keeping it consistent, but does not have to

# Protocol choices
What information is disseminated?

    Active (the operation): Send the command (e.g., "Add 5 to X").

    Passive (the resulting state): Send the actual new value (e.g., "X is now 15").

    Notification: Send a simple "I have been updated" ping.

Which party takes the initiative?

    Push-based (Server-based): The server pushes updates to replicas.

    Pull-based (Client-based): The replica/client pulls updates from the server.

What communication primitives are used?

    Unicast (one-to-one), Multicast (one-to-many), or Gossiping (epidemic spreading).

When is completion reported to the client?

    Immediate: As soon as one server gets it.

    After atomic execution: When all servers get it (e.g., Two-Phase Commit).

    Quorum-based: When a majority (e.g., 3 out of 5) get it.

(Takeaway: Your choices here dictate the consistency model you achieve).


# Local-read primary-write

writse are centralized, reads are distributed. One server is primary, the rest are backups. 

Read path: Client sends request to closest (potentially backup) server. This repospodn with its copy. Fast, but potentailly stale.

Write path:
1. Client send write to local server
2. Local server forward to primary server
3. Tell backups to update
4. Acknkolwedge update
5. Acknolowege write complete to client

# Local read local write
Primary status is now per data item, and can move.

Read path: same
Write path:
1. Write request to local server X
2. Move item x ot new primary X 
3. Ack write completed to client
4. Tell backups to update
5. Ack update

during the handoff, both the old and new primary server may receive writes. It needs to guess if it can write. 

## Quorum based protocols
$N$ replicas

Do a mojority vote for operations. Make read and write quorums $N_r$ and $N_w$ s.t.

1. $N_r + N_w > N$: prevents read-write conflicts: if $N_w$ have the latest write, and read from $N_r$  nodes, at least one node reads the latest value.
2. $N_w > N/2$: prevents write-write conflicts: A majority is needed to commit a write, so two concurrent writes cannot both get a majority.
