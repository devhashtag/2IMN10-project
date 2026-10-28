# Interaction styles

OSI reference model drawbacks:
- focuses on message-passing only, not real-time data or signalling, tunneling (IP - inIP is hard, beacause you have two layer 3's)
- Often has unneeded or unwanted functionality, e.g. the session layer's sync checkpionts, recovery (restore from latest checkpoitn afer reboot), etc.)
- Violates access transparency (applicatiaons must be aware of the fact that a call is e.g. to a remote machine).

Transport layer provides the communication facitilities for most distributed systems.

## Middleware layer
Can be viewed as reusable expendael set of services that are comonly needed by many applications:
- rich set of communicatoin protocols
- (un)marshaling of data, for integrated systems (taking data and makting it suitable for transmission in a message)
- naming protocols, for easy sharing of resources
- Security protocols, for secure communication
- Scaling mechanisms, e.g. for replication and caching

Session, Presentation and application layers build services on top of transport facilities via a programming interface (e.g. sockets) and use the OS's network stack. I.e., transport and down is handled by OS, session and up is handled by middleware or application.

### The Hourglass problem
N applications must bne able to tlak over M different phsyical channels. That's $N\cdot M$ different drivers to write. When using a transport prtocol like TCP, that reduces to $n+m$ (each app uses TCP, TCP has a driver for each connection type,.e.g bluetooth or wifi).

Applications needs are diverse, and different apps cannot understand each others TCP packets. Middleware can provide a similar solution as TCP, by providing a single layer applications can talk with, instead of applications having to learn the working of other apps. It's hard to define a generic middleware, so middleware libraries exists for many application domains. An example is RPC.

Interaction style frameworks typically come with a middleware library.

### Tradeoff
Provide only comunication service (a), or also network interfaces (b)

Case (a): merely provides message passing
- just IP-convergence
- application maintains it specific protocol
- no sharing of logic

This is useful for teams that impelement their own specific application level protocol, but can use middleware to abstract over e.g. differences in how OS handles IP networking.


Case (b): middleware provides advanced services
- Services define the application independent protocol betwene middleware layers.
- Sharing of standard functions: direct access to remote service, supports thin-client approach. For example, RPC is such transparent direct access.


## Communication types

Transient vs Persistent

> With transient communication, the communication server discards the message when it cannot be delivered.

> With persistent communicationk the communication server stores a message as long as it takes to deliver it.


Async vs sync

> In asynchronous communication, the sender continues immediately after it has submitted its message for transmission.

> In synchronous communicaiton, the sender is blocked until its request is known to be accepted.

synchronziation occurs at request submission (middleware confirms request is made), request delivery (middleware confirms the request is being delivered) and after processing by the remote server (response/acknowledgement is given to middlwawre and forwarded to client).

# RPC
Observation: well-engineerd procedures operated in isolation. No reason to not execute it on a separate machine.

Communication between caller and callee can be hidden using procedure call mechanism

Basic working:

A stub is a class that implements the remote interface. The client makes a call on the stub object as if it were a normal object. The stub object builds the message, sends it over the network to the server. The serverside's stub (skeleton) receives the message, unpacks it, performs the desired call on the real object, and messages back the result in the same way.

### Concerns
- __Scalabity__ many clients calling
- __Reliability__ increase dependence, independent failure of servers call for a more eloborate solution (load-balancer with multiple servers?)
- __Performance__

Performance and latency must be taken into account. E.g. an operation to look up a filename, and then read it, should be implemented as one call instead of two, so that the latency is experienced only once.

Elements of realization:
- Parameters are marshalled for transmission
- Definiton of the protocol
- How to deal with parameters
- Hwo and when to bind client and server, discovery
- Semantics under (partial) failures (is the call ran at most once, at least once, or precisel once (difficult))
- Synchronization


### Parameter passing
Usually, no reference or global parameters. This is lakc of transparency

Reference parameters: defaults to pass-by-value bacause of the address space problem.
- Address space problem: memoery adress of remote machine is not valid on caller's machine.
- Copy/restore:
    - copy the in-parameters instead of using the reference, and restore the out-parameter on the client. Inefficient for large data
- Remote reference mechanism:
    An additional RPC can be used to request interactions/modifications on the remote (now client-side) object. However, what about concurrency? A solution is to send the code to the data, so sending a function that is executed near the data, instead of the other way around.

RPC are usually useful when they use implici state (file server) through library calls or use global refereces like URI's. IT breaks the OO oriented model

## Variations on RPC
Traditional is sycrhonous, client waits for repsonse.

### Async RPC
client continues after remote call acknolowdgement. This is used when the reply is not needed. In a one-way RPC, the ack/acceptance isn ot even awaited, the called ocnitnues immediately.

### Deferred sync RPC
Async, with need for reply. Client makes request, and waits for accept, then continues like in async. The server then makes a request back when thre sult is available. So, the client must also be a server.

### Multicast RPC
RPC are sent to multiple servers or a server group. Depending on the applicaiton, the responses are merged or not. E.g. for a fault-tolerant system, the requests are identical and only hte first succesful reponse is used. When the requests differ on inputparamterse, the results may be merged.

Client applicaiton may be unaware that of the fact hte an RPC is actually being forwarded to more than one server.

Both issues can be hidden from the client stub, e.g. when using a mulicast transport address for fault tolerance.


### Summary
(Standard) RPC is tightly coupled sync system.
- Functional couping: caller wants the result
- Temporal coupling: callers waits for reply (blocks)
- Referential coupling: caller has a  reference to a remote procedure (callee)


# Sockets
Sockets are connection endpoints abstraction over the transport layer. It support TCP and UDP. Using TCP it has the following qualit of service properties:
- Transient (temoprary), discrete, connection oriented, buffered, reliable
- Reliable against packet loss, duplicates and reordering, is traded for delay.

Basic interface: just send/recieve

UDP is also basic, but does not have the reliable property.

using transport level address usually (ip+port). Servers binds,listens(max number of pending connections),accepts connections. Clietn sconnect and send/reciveo nly.


## ZeroMQ
Alternative to raw sockets. It uses two sockets

- One for sending messages (at process P) and
- one for receiving messages (at process Q)
- All communication is async.


ZeroMQ can employ request-reply, pub-sub and pipline patterns. the pipeline pattern is like a queue. A pocess can push assuming other processes want to pull, other processes can pull.

# Message oriented Middleware
MoM aimst at high-level persitent async communication:
- processes send each other messages, which are queued
- sender is not blocked waiting on reply, can continue
- middleware often ensures fault tolerance


## Message queing systems
Async persistne communication through support of middleware-level queues.

Operations:
1. PUT: append a message to a queue
2. GET: block until a mesage is in the queue, and remove it
3. POLL: return immediately if a message is in the queu and remove it if it is. Don't block.
4. NOTIFY: Install a handler to be called when a message is put into the queue

### Queue managers
General architecture: each device has a queue manager QM. It can only put thing in and get things from the local queue. QM's need to route messages, as such queus need a contact address (for transparency) asociated with (host,port)-pair, the mapping must be available to all QM's. For scalability reasons, it is uncommon that a QM maintatins list of all other Queues contact addresses and the associated transport level host,port pairs. They function more like routers, forwarding message to the next QM/hop. So they form an overlay network.

### Message brokers
- MQ systems assume common emssagin protocol: all apps agree on message format.
- Broker handles app heterogenigut in an MQ system.
- Transforms incoming message to target format
- Very often acts as an application gateway.
- May provide subject-based routing, like pub-sub.

A broker is just another applicaiton in queueing system, so it's not integral. It has plugins to convert format x to y, so it has an at most (2*)n*n table of conversion routines. It effecitvely encodes informaiton of the messaging protocol of different applications. An application gateway is simply a "frontend" for the application. So other apps do not have to know where it is, how many instances run, etc. They drop a message at the front door, optionally indicate which topic it is, and the broker decides where it goes.

- Routes message basd on topic,queue, or routing rules
- Protocol gateway: converts message formats
- Decouples apps
- Pub-sub support: msg can be forwarded to multiple subs
- Adatper/plugins: format conversion is dynamically adaptable using plugins.


# QoS techniques
- __Buffering__ smooths delay variation (but introduces consistent lag)
- __Priority routing__ reduces latency and jitter for prioritized packets
- __Flow control__ route (re-route through a route that has more bandwidth) and rate (slow down sender if receiver can't keep up, or network is congested)
- __Resolution and quantization scaling__: pixels per frame (usually video, not audio ), or bits per pixel (usually audio, video) can improve throughput.
- __Quality modes__: Signal is divided into multipe streams, mode decides which streams are sent. e.g. video resolution. Can encompass multile QoS aspect, e.g. high framerate AND high quality.

QoS requirments can dictate the type of communication that is used. Most distributed systme rely heavily on transport protocols to offer their functionality.


# RabbitMQ:

1. Publisher send message to an __exchange__. (Type must be specified)
2. Exchange __routes__ the message bassed on attributes, e.g. routing key, type.
3. __Binding__ connect the exchange to queues, a message is delivered to a queue according to the routing rules.
4. A queue holds the message until it is consumed
5. A consumer processes the message


