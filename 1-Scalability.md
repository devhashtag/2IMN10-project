# Scalability

> The ability to handle increased workload (without adding resources to a system).

> Scalability is the ability to handle increased workload by repeatedly applying a cost-effective strategy for extending a system's capacity.


Plays a major role in SLA of applications.

Several scalability aspects of the system should be considered.

Desining elastic systems is challening.

Scale in dimensions:
- Size (number of users/processes)
- Geographical spread (max Distance between nodes)
- Administrative domains (adding more admin. domains)

Types:
- Vertical (e.g. adding resources to a server)
- Horizontal (adding more servers)

Load balancing: Hardware or software that distributes incoming requests among available servers

Microservies: The application is organized in loosely coupled, fine-grained microservices that communicate through lightweight protocols. Allows scaling up/down per bottleneck instead of the entire app


## Size scalability

Root causes:
- Computational capacity
- Storage capacity
- Network capacity

> Utilization U is the fraction of time a service is busy, or $\frac{\lambda}{\mu}$ in poison process

## Geographical scalabiilty

Problems:
- Moving from LAN to WAN introduces **latency** and **reliability** problems
- Lack of multipoin communication, search broadcast cannot be deployed -> Use separate naming and directory services


## Administrative scalability

Problems:
- Conflicting plicies concerning usage (payment), management and security. (E.g. sharing expensive resources between domains in a computational grid)

Exception: several peer-to-peer networks like file-sharing systems (bittorrent) and peer-assisted audo streamign (spotify)


## Tactics
- Hide commuincation latencies (but how?)
- Async model
- Separate handler for incmoing requests
- Partition data and computations across multiple machines
    + move computations to clients
    + Decentralized naming services (DNS)
    + decentralized information systems (WWW)
- Replication and caching: make copies of data available on different machines
    + Replicated file servers and db's
    + Mirrored websites
    + Web caches (in browser and proxies)
    + File caching (at server and client)

Replication is easy, except for when modifications are applied to a replicated resource. It needs to be in sync with the other caches/copies, thus a **global sync** mechanism is needed. If **inconsistencies** are tolerated, the need for global sync is reduced but this depends on the application.

## Measurements

1. Scaling of system size has **no effect** on the **sequential part**
2. Scaling of system size has **linear** effect on the **parallel part**

### Amdahl's law

Keep problem size fixed. Problem size does not change if more resources are available, so $s$ does not shrink. It becomes a bottleneck, hence the horizontal asymptote

> Speedup = $\frac{s+p}{s+p/N} = \frac{1}{s + p/N}$

Where $s$ is the fraction of time spend on the serial parts of the program, $p$ is the faction of time spend on parallel parts of the program and $N$ is the number of processors.

Lesson: if tuning a variable has neglible impact, move attention elsewhere (to another system variable)

### Gustafson's Law

Observation: parallelism increases when the problem size increases. Execution time is fixed, but the problem size increases. Usually, the parallelism is then higher than the serial workload.

> Speedup = $S + (P\cdot N)$

Where $S$ is the fraction of serial machines, $P$ the fraction of parallel machines and $N$ the number of processors.

Lesson: Use additional resources to solve a larger problem in the same time (scaled speedup), rather than the same problem faster — that's where near-linear gains come from. (3D rendering: load assets is constant, but rendering pixels can grow indefinitely with higher resolutions)

## Elasticity

> Ability of a system to adapt to workload changes by provisioning or de-provisioning resources in an automatic manner.

Needs an adapataion mechanism, e.g. auto-scaler which may use a user-configurable policy


Scalability of a prequisite of elasticity

### Aspects

Timing:
- percentage of time the system is over- or under-provisioned, or in perfect state.
- Frequency of adaptations
- Depends on adaptation policy

Latency:
- Time to bring up or bring down a resources
- Depends on adaptation mechanism

Accuracy:
- The relative deviation of amount of _allocated resources_ from the _actual resource demand_, expressed as average over time.
- Depends on the allocation granularity and adaptation policy

