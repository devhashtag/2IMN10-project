# Week 1

#### Definition of a distributed system
> A distributed system is a colleciton of autonomous computing elements that appears to its users as a single coherent system.

### 1.1.1 Integrative view
Connecting existing networked computer systems into a larger system. 
Example: services running on a system must be made available to users and applications that were not thought of before. The system is combined with another system, so that all required parties can access it.

### 1.1.2 Expansive view
An existing networked computer system is extended with additional computers
- expanding the system close to the places where the resoruces are needed
- improve dependability, e.g. if one computer fails

## 1.2 Organizations

### Centralized
A computer system where all computation, control and data are managed by a single central authority.

### Decentralized
A distrubted computer system where all computation, control and data are distributed across multiple interconnected computer systems without a single central authority. This can e.g. be multiple centralized systems connected.

### Distributed
Systems in which computation, control and data are distributed across multiple interconnected computer systems that coordinate to achieve a common goal.

Main characteristics are:
- No global notion of time
- No global notion of state
- Heterogenous resources (from high-end server to low-powered sensor)
- Network communication
    - wired or wireless
    - latency can vary from microseconds to seconds
- Independent failures, each element can fail at any time

## 1.3 Design goals

### Resource sharing
Users share and access remote resources, .e.g cloud-based files, peer-to-peer streaming and shared mail services.

### Distribution transparency
A high distribution transparency means to hide the fact that the processes and resources are physically distributed across multiple computers.

This can be realized by a **middleware layer** that sits between the applications and the underlying platforms, tying together distributed applications, databases or devices. There are different types of distirbution transparency.

1. **Access**: Hide differences between data representations and how objects are accessed.
2. **Location**: Hides where an object is located.
3. **Migration**: Hides that objects may be moved to different locations.
4. **Relocation**: Hides that objects may be moved to different locations _while in use_.
5. **Replication**: Hides that objects may be replicated on different locations.
6. **Concurrency**: Hides that objects may be in use by several independent users.
7. **Failure**: Hide the failure and recovery of an object. It should be invisible to users and applications and be recovered from automatically. This failure masking is really hard and sometimes impossible.

### Scalability

> Scalability is the ability to handle increased workload.

> Scalability is the ability to handle increased workload by repeatedly applying a cost-effective strategy for extendings a system's capacity.

### Openness
A system is open when it 
- offers components that can be easily used by or integrated into other systems. An open distribuyted system itself will often consist of components that originate from elsewhere.
