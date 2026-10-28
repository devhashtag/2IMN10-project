# Naming

- Name serves as reference to entity
- Name space: set of valid names - sepc/alg as how to validate/generate names

## Motivation
- Identification
- lookup - location transparency
- humand-friendliness

# Entities
> Entity is a thing that is referred, e.g. resource, device, network port, queue,
- Identifiers are the namespace for entities
    - An identifer referse to at most one entity
    - each entity is referre to by one identifier
    - an identifier always refers to the same entity (it is never reused)

Examples:
    - MAC address is an identifier for network card 
    - IBSN nubmerof book

> Access point represent a place where an etityt can be accessed

> Address represents a name space for access points

> A name system is a name-to-address binding

SImplest form is a big table. Often in form of distributed database

A name service answer queries for attributes of the name entity
- typically addreses, but not always
- Name server may only respond forp art of the name space, its domain of authroity (e.g. DNS ns)
- May invoke other server to answer the query (recursive dns)

## Binding
Establishment of, relationship between a reference/name and the referred object (access point or another name)
- Symbolic: referred objerct is itself a similar reference
- Aliasing: Several names for the same object 

## Naming types
- Flat naming: identifiers are random bit strings
- Structure-based: composed from simple readable names, e.g. tree

## Resolution
Given a name, retrieve its referred object (read: access point)

1. Structured resolution: Follow structure and levels in naming hierarchy, e.g. given DNS name, find transport (IP) and then MAC
2. Flat resolution: flat naming, no hierarchy. E.g. reverse ARP: what is the IP of this MAC?
    - Given identifier, determine access point
    - Reverse DNs: what is the DNS name of this IP address
3. Closure mechanism: knowing where and how to start resolution.


## Search mechanism

- Broadcasting is imply asking everyone, limited by phyiscal scope (e.g. ARP)
    - Broadcast ID, request entity to return its current address (AP)
    - Can not scale beyond LAN's
    - Requiers all processes to listen to incmoing location requests (n^2)
    - Example: ARP asks everyone who has this IP address, owner respond with MAC.
    
- Multicast is asking multiple  others e.g. via a multicast address or flooding. Anyone that listens, can respond. Limited by logical scope (e.g. searching in p2p overlay networks). 
    - flood-based Multicasting means that every node in the overlay must get it, so it's similar to broadcasting.
    - on msg m: forward to all neighbours, except sender. Keep track of received message and discard duplciates
    - Eventually all nodes get the message

- Table lookup, hashing or tree-search
    - Hashing in side repositories
    - Distributed hash tables
    - DStribiuted tree like reposities

## Flat resolution

How do two parties that don't know each other meet?

> Service discovery is the action of finding a service provider for a requested service
 
Client directly contact known access points for server access, based on globally accepted list of services.
    - E.g. port 21 for FTP, 23 for telnet, 22 for ssh

Example: Netflix Eureka: managin a registry ot f SErivce Instances. Providing a service menas to register, de-register and query Instance with the registry. The eureka server registry propgates to other euroka instances (server or clients ).


### Broadcasting
- High overhead: no duplicates, but many redundant forwardings.
- > Flooding trades simplicity for high message overhead
- #messages = $2|E| - |V| + 1$

We can model the overlay network without topology by a random graph. Each edge hsa $p_{edge}$ probability of existing. So, we have $2|E| = p_{edge}\cdot |V|\cdot |V-1|$ as expected value.


### Forwarding pointers
When an entity moves from A to B, it leaves a pointer to B, in A. That way, an entity can be tracked down from A by following pointers. Drawbacks include:
- Mobile entities give rise to long chains, making it exepsnvie to look up
- Previous locations must maintain pointers
- If one pointer gets lost (broken link), chain breaks (not fualt tolerant)

### Home-based approach
Let a home keep track of the location of the entity. Outside devices will always contact the home (registered in e.g. DNS) in order to communicate with the device on its original IP. When the entity moves, notify the home. Client contacts home using normal resolution, if the entity is moved to a foreign location, the foreind address is given. In mobile IP (hybrid model), the packages are tunneled to the curent locatin of th enetity as well, until the client contacts the entitty/host directly.

## Distributed Hash Table
- Data does not fit on single server, horizontally scaled across servers.

- Use a hash module $N$ to distribute keys across $N$ servers.

If a server fails, keys must be redistributed. How? Consistent hashing.

### Consistent hashing

Assume you have $N$ object keys, not necessarily numeric. Use a hash space of $k_0$ to $k_{n-1}$. Connect ends so its a ring. Then each hashed object gets a plce on the ring. The trick is to hash (working) server's names, so they too get ap osition on the ring. To find which server a object key belongs to, just walk the ring clockwise to find the first server. This way, you can add/remove servers without potentially much redistirbution of keys, since moduleo is not used.

### Chord: distributed storage
Nodes are organized into a ring,. of size $2^m$.

$m$ depends usually on the hashing algorihtm, its 128 or 160 often.

- Nodes have $m$-bit identifier
- Entity have $m$-bit keys
- entity with key $k$ falls under node with smallest $id \geq k$, so the clockwise neighbour.

Naive approach: Let p be node id. keep track of succ(p) for each node. Forward the request for key unless pred(p) < key <= p. Travels half the ring on average.

Bettere approach: use finger table.
Finger table, one per node $p$, stores $s\leq m$ entries, with $FT_p[i] = succ(p + 2^{i-1})$. So it contains a list of nodes with roughtyl exponentially increasing distance between them.

To look up key $k$, node $p$ forwards the request to node $q$ with index $j$ s.t. $q = FT_p[j] \leq k < FT_p[j+1]$. (or $q = FT_p[1]$ if $j k < FT_p[1]$.

New node joins:
- Locate successor,
- update succ and pred pointers
- transfer keys from successor
- update finger tables
    - other nodes need ot update their finger tables to include n
    - chord uses stabilization protocol to propagate these updates incrementally over time. (on each node, as background process, FT is updated) node $q$ contacts node succ(q+1), and requests pred(succ(q+1)). If this is not q, then a new node is in between and needs to be added. Check also if the new node has q as pred, if not, another node is inserted. Keep doing until q is returned.

Deletion of node:
- beflore leaving, n transfers keys to successor
- update succ and pred points, n tells its pred to point to its succ, and its succ to point to its pred.

## Problem
Logcal organization of nodes inthe overlay may leadt oerractic message transfer in the undelrygin network, if a node ans its successor are far apart.

Solutuions:
- Topoltoy-aware node assignment, when assigning id to node, make sure that nodes close in id are also close in the network. Can be difficult.
- Proximity routing: maintain more than one possible succesor, forward to closest. E.g., maintain a range of nodes instead of one, and pick the closest node $q$, while maintainnig that $k \leq q$
- Primixiry beighboiurt selection; (not in chord) if oyu have choice of neighbour selection, pick closest one.

# Structured naming

Graph, edge labels are partial names leafs are access point or addresses.

Directory nodes are access point to dir sturcture, e.g. with label "/". HAve list of child nodes.

If root nodes is in path, then tis absolute path. Otherwise it's a relative path.

Examples: file systems, internet addresses (a.b.c.d), DNS, FTP url (combiens several namings).


## joining name spaces
- Direct concatnion as in some url schemes: <domain>/<path>
- Use name of system as prefix like systemA:/home/..., resolve in the new root
- Symbolic linking: Store the reference to the actual contents in second name system, as the referred-to-object: e.g. /domains/private_html continas /public_html.
    - Siimlar to mounting, but less dynamic, since the reference is stored int he file itself.


## Mounting

To tell the resolution procedure, that from point $p$ onwards, a different resolution should be used.
- Modification of resolution procedure
- Closure from p onward stored with p, in a table at the resolver (usually OS)
- transparent resolution: not noticable



# DNS

Global layer: root and  TLD servers, anycast comms with client
Administration layer: Nodes managed by a single administration (e.g. tue, org, company, whatever.)
Managerial layer: not part of DNS. Frequencyly changeing local resources, part in url after DNS name (so /home /about /file/on/this/path.txt)

> A domain is a subtree of the naming space

> A zone is a specific portion of a domain for which a particular name server is authorative 

A zone does not include sub-domains, they are spearate zones.

Zones partition the tree.

Extra-functional properties:
- Modification by different adminstrations
- Extreme robustness
    - Even when breaking apart the distributed system
- Scalability
    - Geographical srpead
    - number of machines
    - number queries
    - query response time
    - scalability criterion: constant on average, at worst an occasional spike
    - Appliec tactics
        - namespaces partionition (distributed database), server replication, caching

## Nameservers
Name server serves a certain _zone_
- Authorative or cached answer
- Authorative server for ah ost always maintain the records of that host

Root server: are tuahotitave for root and tld

Name servers are configured with addresses of root servers

Nameserver for a domain
- know child domains
- are typically replicated (primary initialied from file and secondary servers (are synced from priamry))

DNS is made avialable to apps as a middleware service
- typically, a library (resolver) that calls upon a configured name server
- That server often uses BIND (berkely internet name daemon)


## Distributed resolution

Iterative: Ask root server for (nl, tue, ftp), returns .nl NS IP and (tue, ftp), ask .nl NS, returns tue.nl NS, ask tue.nl, etc.

Recursive: The root server asks the nameserver directly, and that in turn the next namserver, until the auth ns is reached. It returns all IP's of all steps. So the root receives the ip of .nl, tue.nl and ftp.tue.nl. And can cache this.

the cache for iterative is done differently. It is cached by the iterative server. Hence, a local name server is used to improve caching.

Tradeoff
    - recursive servers caches every step.
    - Locality in name is expolited: If a.b.c is queried, and someone else asks for x.b.c, b.c. is already known
    - repeated queries do not enjoy optimization
    - in Iterative case: the number of commuincations is the same, but the sum of RTT is not. (some NS lower in the three may be same, so lower RTT when recurisve)


## Record
A record contains:
- Domain name
- TTL
- Class (mostly INternet)
- Type
- Value

Types of records:
- SOA, Zone, info on represneted zone
- A host, IP adress of the host this node rperesnts
- MX Domain, contains mailserver handle for incmoing mail
- SRV Domain , refers to a server for some service    
- NS zone, refers to a name server that impl,enetsn the zone
- CNAME node, symolbic link with thep rimary name of the repsrentd node
- PRT host, canoncilca name of the host
- HINFO host, contains informaiton on the host (e.g. cpu, network card etc)
- TXT, any, conatins nay entity-sepcific informatino considered useful

There are 13 root servers, many are replicated (f has 46 sites as of 2009). This is beacuse only 13 lists of NS records and A records can fit inside a 512 byte DNS packet response, (when requesting who serves the root '.').

## Scaling
- Response time depend on depth of serach
- Global servers get lot of requests -> limit traffic per soruce
- Strategies
    - Replicate
    - Closure mechanism: start at local server that uses caching
    - Addresses are assumed not to change frequencly (e.g. 24H TTL for TLD). highly Mobile devices that cross domaisn do not benefit from caching.


## REmarks
DNS with DHT impl,emetnaiton is possible. Can be done fro any naming system. However, structure would be lost.

DNs is based on key, which is domain name.

Lookup based onattribute value (includign searching and wildcards) can be needed. A set of (attribute, value) recrods is called a directory. Service term is direcotyr service, as opposed to naming service.
