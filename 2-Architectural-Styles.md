# Architectural Styles

> Software Architecture is the logical organization of a distributed system into software components

The realization of distributed system happens by instantiating and placing software components across multiple machines.

## Purpose of architecture

Communication / understanding (to define the scope of the system)
- With stakeholders to see if the right system is made
- With developers/teams to guide the construction
- With users/customers to provide insight in how to use the system

Analysis
- To evaluate candidate architectures on an abstract level
- To make design decisions

Synthesis
- Blueprint for the overall structure
- Basis for detailed design by developers


## Requirements

A system **stakeholder** is anyone who should have some direct or indirect influence on the system requirements. For example, end users who will interact with the system, or engineers who are developing/maintaining similar systems, business manager, domain experts, etc.

Requirements describe what the system should do:
- Inlcude services provided by the system
- Define constraints on how the system operates
- Reflects needs of customers
- Shaped by intended purpose of the system


### Functional requirements
These describe the expected functions and behaviour of the system _from the users perspective_.
Define the services the system must provide, may also specify what the system should not do to avoid unintended behaviour. 

### Non-functional requirements
- Define constraints on the system services or functions rather than the services themselves. Can include timing constraints, development constraints (use specificmethod/tools) or standards compliance.
- Typically apply to the entire sytem
- Affect system quality attribute (-ilities, e.g. accessibility, usability, repsonsiveness, security, etc.)
- Affect the choice of architecture



## Style

Formatted in terms of:
- Components with well-defined interfaces
- How they are connected (wiring, structural connectivity graph)
- How data exchanges between components
- How they are jointly configured into a system (systemwide instantiation, which node/process/container each component lives in, lifecycle, resources, etc.)

A combinatino of architectural styles is common
Client-server is centralized and widely used.

> A component is a modular unit with well-defined interfaces

> An interface is a shared boundary across wihch two or more separate components of a system exchange information

> A connector is a mechanism that mediates communication, coordination, or cooperation among components. (e.g. RPyC for remote calls)

## Service
A contracutally specified overall functinoality of a component. E.g., an IO file system is a compnen,t the service is that users can store and retrivee files. Service interface is an API. Responses can be events or callback ( autonomous )

## Protocol

> A protocol is a formal set of rules that dictates how information exchange, as well interaction between entities should take place.

The rules specify:
- Message format
- A finite state machine
- Timing constraints
- Other quality peropties, if any

##  Description of style
Motivation
    - Which extra- or non-functional properties are supported?

### Client-Server style
Client makes request to server, server responds

#### Structure

It's dynamic, connection pattern arises at runtime with many-to-one
Serverl ocation may be determined at deployment time
Server may be a distributed entity

#### Behaviour

Clients find server access point through __discovery__: finding the server's location through a certain mechanism.

Servers are passive, provides services upon requets from clients. Does not know clients. Handles data access and integrity

#### Rules

Clients are active and initiate actions.

No conneciton among clients, limited or no state on the server per client

#### Motivation (EFR/NFR)
- __Cost reduction__ by sharing some localized resources
- __Latency__
- __Modularity__ decreases dependencies


#### Vocabulary

- Client, server (building blocks)
- Request, reply, session (interaction mechanism)
- Server discovery, access point


#### Weaknesses

- Single server can be a performance bottleneck
- Single point of failure
- STate servers scale poorly, because it requierst memory space and synchronization to keep the status of client on the server side up to date.


#### Example: Dropbox Content-addressed blocks
- Content adressing: Each block is identified by a hash of its content, not its position.
- Efficient sync: Only blocks missing from the server need to be uploaded
- Deduplication: identical blocks can be stored once, unless per-user encruptiopn makes them different


### Layered style

#### Motivation
- Modifiability: Separation of concerns makes the layers easy to change, unless you change the interfaces
- Modularity: Independent development and independent evolution per layer.

#### Vocabulary
- Layer, Module, Service, Interface (building blocks)
- Allowed-to-use-relation (interaction)
- Call, downcall, upcall, call-back-routine, handle (interaction)

#### Structure
System is organized in independent layers of abstraction. Each layer is a group of modules offering a coherent set of services

Layers are determined at development time.

Types:
- Strict layering
- Non-strict (layer briding allowed, i.e. calls to $N-m$  with $m \geq 1$)
- Sidecar: for modules adressing crosscurtting concerns, e.g. a util library. Acceissable from all layers

#### Behaviour
- Objects in layer $N$ call upon the API if objects only in layer $N-1$. Layer $N-1$ is regarded as the single service provider of all lower functionality
- Layer $N$ registers call-back routines for events with layer $N-1$. Layer $N-1$ invokes the callback referred to by the _handle_.

### Rules


- Layer $N$ may use onl lower layers. In case of strict layering, only layer $N-1$ may be used.
- for modules within a layer there are no "use"-restrictions.
- Control flow can be both sync and async (event, callback)

An upcall is done by layer $N-1$ to layer $N$ by invoking a handle given to $N-1$ by $N$, at an earlier time

#### Weakness
- Each layer adds additional complexity, additonal code and longer calling sequences.
- It is an anti-pattern in a setting that is too simple

#### Example: OS, or Network stack
User clicks on a file, application makes reqeuest to library, library makes OS call (e.g. posix), kernel uses hardware drivers to fetch the file.


#### Multitiered centralized system architecture
Three layers:
1. Application-interface layer: units for interfacing to useres or external applications. E.g the frontend in a web app
2. Processing layer: functions of an application, i.e. without sepcific data. E.g. the backend in a web app
3. Data layer: contians the data that a client wants to manipulate through the application components (e.g. a database or file system)

Single-tiered: mainframe-type, everything from data-backend-frontend on one machine
Two-tiered: client/server configuration
Three-tiered: each layer on a separate machine


##### Two-tiered
Multiple distributions:

Thin client: e.g. a terminal app only has part of the UI layer, the rest is server side. Also possible to have the full UI on the client side, e.g. Google Docs
Thick client: e.g. MS Outlook, has UI, Processing and Data layers. Backend only has part of data layer

##### Three-tiered
Latency penalty: total latency = client-app latency + app-data latency


### Peer-to-peer 

#### Motivation
- Cost reduction through resoruce sharing (e.g. files)
- Concurrency: more peers usually means faster task completion
- Scalability: horizontally scalable, compute/storage is distributed across the internet
- Fault tolerant: Robust to faults or attacks, due to the decentralized nature.

#### Vocabulary
- Peer, super peer (building blocks)
- Overlay, distributed hash table (building blocks)
- Community (of peers), peer discovery
- Decentralized
- Application-level multicast


#### Structure
1. Peer finds community access point, most likely a super peer
2. Joins community
3. Provides passively services to other peers
4. Actively uses services from other peer
5. Collective services of peers result in new functionality e.g. file sharing

#### Rules
Peer
- Symmetric in fcuntionatliy and contribution
- Can discover, reach and cooperate with all other peers
- Can be both passive and active

Super peer
- Contributes extra resources to the comunity (extra services, like directory and discovery)

#### Structured P2P
Peers connect to super peer to get the service. Nodes are organized in an overlay that aderes to a specific deterministc topology. An exmaple of this is centralized P2P


Centralized P2P:

1. Peer connects to super peer, requests file x. Super peer responds, file x is at peer y.
2. Peer connects to peer x to download file x.

An overlay is a logical virtual network on top of the physical network. An example is the distributed hash table: It dictates the _topology_. E.g. is the network a ring, or a tree-based network? It also dictates the routing. If you want a file,you can find the node in a small number of hops, you don't have to flood the network to ask all peers which has the file. 

#### Unstructured P2P
Each node has list of neighbors. Overlay network is a random graph. Access to some file is gained by flooding or a random walk.

##### Flooding
1. Node u passed request to all neighbors
2. Request is ignored if the node has seen it before (to prevent recursion)
3. Search is limited by TTL or maximum number of hops

Peer that has the file, responds to all requests if multiple arrive, since they might be initiated by different peers

#### Random walk
Instead of flooding, a node passes the request to a random neighbor. If it does not have the file, pass to another random neighbour and so on.


#### Weaknesses
- Security, peers come from differetn domains of authority
- Super peer is single point of failure
- Peers are transient; they come and go. Difficult to maintain consistent quality levels.

### Publish subscribe


#### Motivation
- Modifiability (flexibilty, extensibility): decouping data producer from data consumer
- Scalaibiltiy: multiple publisher and subs can bne added without significnat reconfiguration
- Concurrenty: allow multiple publisher and consumers

#### Vocabulary
- Pulibhser subscriber, broker
- Subscription, topic, notification, event, filter


#### Structure

1. Pbulishcer register to broker, or are discovered
2. Subs find brokers and send subcriptions to certain topcis
3. Brokers forwards notication to subscribers that have subscribed to the topic
4. Broker _can_ be emitted, if subscribers register dircerlty to publisher. Reduces communication, but no refential decoupling

Strong sepearation between processing (subs, mabye pubs) and coordination (brokers).

> Coordination encompasses the communication and cooperation between processes


Temporal coupling: situation where components must operate in sync with specific timing or deadlines.

Referential coupling: dependencies between components/modules based on shared data or references, e.g. two devices need each others IP adress.

#### Rules
- Every subscription relates a topci to a sub
- Publisher can have multiple subs to the same topic
- Notificaiton goes to all subs, but subs may habe receive policies (filters?)
- If brokers exist, they decouple the subs and pubs

#### Weakness
- Larger and fluctuating latency
- Not suitable for hard real-time constraints due to this


