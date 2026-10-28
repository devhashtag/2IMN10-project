TODO: Raft

# Fault tolerance

> The ability of a system to deal with faults

> Graceful degradation of service: The critical parts of the system will continue to work, non-essential parts will temporarily be disabled

## Metrics
- MTTF: Mean time to failure
- MTTR: Mean time to repair
- MTBF: Mean time between failures: simply MTTF + MTTR

Quality attributes:

- Reliability: continiuit of correct service (expressed as length of time)
Is measures by Mean Time To Failure (MTTF)

- Availaibilty: readiness for correct service (expressed as probability)
Measured by MTTF (MTTF+MTTR)

## Definitions

> Failure: not meeting the specification.

Externally observable deviation from the epxected correct behaviour, e.g. a crashed program.

> Error: System state that may lead to failure

Does not need to be externally observable, system may recover before leading to failure. e.g. becauseo of a bug, or tmeporarilty losing internet connection

> Fault: cause of an error

Can be the failure of another (sub)system, e.g. a bug in a sloppy program


### Fault description

Origin:
- Internal: at any of the system components
- External: or in the environment

Reasons:
- Accidental: bugs, wear and tear
- Malicious: attack on systems (byzantine failure)

Durations:
- Transient: exist and dissapear
- Intermittend: occurds repeatedly, disspaeards in between
- Permanent: remains until repair (fault handling)

> Failure model identifiers the system component where faults may occur. Classifies the type of faults that can occur, and defines the notion of correct behaviour.

Failure types
1. Crash Failure: Server halts. It works correctly until it halts.
2. Omission Failure: Server fails to respond (Receive omission: fails to receive incoming; Send omission: fails to send outgoing).
3. Timing Failure: Server responds, but outside the specified time interval.
4. Response Failure: Server responds, but the value is wrong or the state transition is wrong.
5. Arbitrary (Byzantine) Failure: Server produces arbitrary (potentially malicious) responses at arbitrary times.

Halting failures
1. __Fail-stop__: crash failures, but realiblty detectable
2. __Fail-noisy__: crash failures, eventuellay realiasbly detectable (e.g. eventua lerror messaga)
3. __Fail-silent__: Omission or crash failures: client cannot tell what went wrong (timeout without error message)
4. __Fail-safe__: Arbitrary yet benign failures (cannot do any harm) (e.g. subcomponent is wrong, but is not essential to critical functions)
5. __Fail-arbitrary__: Arbirarry, with malicious failures

Redundancy for failure masking
1. Information redundancy: add extra bits (error detection or correction) to data, so that errors can be recovered from
2. Time redundancy: Design system so that action can be done again, if something goes wrong. Useful for transient or intermeitten faults (idempotency needed?)
3. Physical redundncy (heailvy used in Distributed Systems): add requipemtn or pcroesse to allow more than one component of ail. E.g. multiple server backends with a load-balacner in front it


## Triple modular redundancy (TMR)

Signals pass through A, then B, then C. If one fails, result is incorrect

The pipeline is copied three times. At each stage, inputs from the three A's, are sent to three voters. Each votger has three inputs and one output. They cast a moajoirty vote, if there is no majority then the output is undefined. The outptus of the three voters are each sent to B, which ahs the same structure, as does C.

# Fault/error detection

1. Acceptance test: check correctness of reults
2. Exceptions: raised when system condition is detected, exception alters normal control flow
3. Watchdog timer: timeinig error when result is not avialalbe in specified time
4. Ping/echo: component issues ping, measrues time till echo. If nothing arrives, means there is fault.
5. Heartbeat: component periodically emits heartbeat message. Other component listsn and meausers the time. Detection happens when _nothing_ is received, indicate sfualt.


# Process resilience

> Resilience is a metric to measure the level of fault tolerance: a $t$-resilient  system can tolerate up to and including $t$ faulty components.

Has theoretical bounds onthe achieveable resilience ($t_{max}$)

Protect against malfunctioning processes by replication and grouping. A flat group has equal workers, a hierarchial group has a coordinater ans workers (e.g. load-balancer and backends).

__$k$-fault tolerant group__:
- Group can mask any k concurrent member failures
- How large much gorup be?
    - with halting failrues (crash/omisison/timeout/response): k+1. No memeber will producei ncorrect reults, so one repsonse is good enough.
    - With arbigrary failures: 2k+1 memberse, so taht correct result can be obtained with majority vote ($k$ liars, so $k+1$ honest needed)

Assumpetions:
- all members are equal
- all members process commandss in the same order

Result: we are sure each process does the same thing


## Consensus
Prerequisite: all nonfaulty processes executes the same commands in the smae order as every other nonfaulty process

Reformulation: nonfaulty group members need to reach consensu which commands to run next

Examples: selecting a leader, commit or abort transaction


Properties:
1. __Agreement__ Every correct process agrees on the same value
2. __Termination__ Every correct process decides some value
3. __Validity__ If all processe propose $v$, then all correct processes decide $v$.
4. __Integritry__ Every correct process decided at most one value, and if it decidse v, someone must have proposed v


### Flood based consensus

$n$ processes $P_i$, work in rounds. EAch process has a set of commands $C_r$  at round $r$. In each round, the set of commands is sent to every other process (n^2 flood). Then it decides using a glboal, dtereministic function ( every nonfault process decides the same thing given same inputs). If a crash is detected, processes that don't have all commands received yet, simply do not make a decision.

More realistic approach: Raft

### Dependability

A component $C$ depends on $C*$ if the correctness of $C$'s bevahiours depends on the correctness of $C*$'s behaviour.


CAP-theorem: Any networked system providing shared data can only provide two of the following properties:
- Consistency: all replicated items appear a a single up to date copy
- Availability: updates which always be eventually executed
- Partioning: tolerant to the partitionig of the process group (e.g. failure of the netowkr among servers). (network split in two or more) -> can't communicate

If partinoning happens, you have to choose:
- Keep available -> updates happen, but consisteny is lost (other part of the network does not get updates)
- Keep consistency -> no updates can happen, system is unavailable, but state remains consistent

Example: bank with two atms. if partininoed, it is tricky to keep avialbility: one can withdraw at both ATM's, and recieve more moeny than they have balance.


Conclusion: In a network subject to communication failures, it is impossible to realize an atomic read/write shared memory that guarnatees a reponse to every request.


# Reliable remote preocedure calls

What can go wrong:
- Client us unable to locat eserver
- request message from client to server is lost
- response from server to client is lost
- server crahses after receiving request
- client crashes after sending request

"Easy" Solutions:
1. Cannot find server: Report back to client
2. Request was lost: Retry

Server might crash before OR after executing the crash. Both require a diffrent solution, but difference cannot always be detected.

Two approeches:
- At-least-once semantics: the server gaurantees that it will carry out an operation at least once
- At-most-once: the server guarantees it will carry out an operation at most once

(?Assumption: Server can identify if two request are the same?)


## Reliable group Communication
Setting: Group G with members, a mesage is sent to all members.

Make distinction between receiving and delivering message.
- Receiving: a message is received in the buffer, either in OS or middleware.
- Delivered: a message is delivered from the buffer to the application.

The middleware can then assure that messages from one sender are delivered in the order that they were sent, if some sequence number is used.

Group communication is reliable when when it can be guaranteed that amessage is received and subseuqently delivered by all nonfaulty group members.

1. Sender keeps a message in history buffer, until it has received an ack from all group members.
2. If a member receives message m, but m-1 has not been received, it sends a NACK m-1, requesting retransmission. Otherwise an Ack m is sent.

Tricky part: Agreement is needed on what the group actually looks like before a received message can be delivered: which members are fault and which are not?

Scalable Reliable Multicasting: only send nacks, not acks. Also multicast them, so other senders don't have to send nacks if they miss the same message.


## Recovery
- Forward recovery: find a new error-free state to continue, e.g. reset to initial state.
- Backward recovery: bring system back into previous error-free state.

Practice: use backward error recovery, requiring that we make recovery points

In distributed systems, this is complicated by the fact that processes need to cooperate in identifying a consistent state from where to recover.

### Consistent recovery state
Requirements: every message that has been received, is shown as sent in the state of the sender.

Recovery line: assuming processes regulary checkpoint their state, the recovery line is the most recent consistent global checkpoint. (by above requirement)

Observation: A message can be sent, but not delivered, in a consistent checkpoint recovery line

## Coordinated Checkpointing

1. A coordinator multicasts a checkpoint request message.
2. when a prticiapnt receives it, it takes a checkpoint, stops sending (application) messages, and reports back that it has taken a checkpoint.
3. When all checkpoints have been confirmed at the coordinator, it broadcasts a checkpoint done mesasge to allow all processes to continue.

Can be improved by only considering processes that depend on the recovery of the coorindater, and ignore the rest. Useful if there are multiple "levels" of coorindators.

# Tactics summary
- Availibility and reliability improvements comprise mainly:
    - Extending the up-time
        - restricint probablit of occurrence of faults/errors (infoermation redundancy physical redundnacy)
        - dealing systemtically with partial failures (consensus, reliable group communication)
    - Restricing down-time 
        - facilitating fast repair/recovery (coordinated checkpointing, finding recovery line)

These are Fault tolerance techniques, they prevent faults from becoming failures.



# Checkpoint storage in Flink

- Flink periodicallte takes persistent snapshots of all the state in everyt operator, and copies them in a more durable storage, like a distributed file system 
- On failure, snapshot is restores, and the application can continue
- The location of storage is defined via the jobs checkpoint storage.

## Snapshot

> A snapshot is a global, consistent image of hte state of a flink job. 

It includes:
- a pointer into each of the data
- A copy of the state from each jobs stateful operators, that results from having porcess all of the evens up to those posision in the sources

Checkpoint: a snapshot taken automaticlly by flink for purpsoe of fault restoration. Can be incremental, and are optimzied for being restored quickly.
SAvepoint: snapshot triggered  anually by used

flink support three guarantees:
- at most once: no recovery, events may be lost
- at least once: no events los, but results may be duplicated
- exaclty once: eacht event affects flinks managed state exactly once.

Exactly once IS NOT only procesesd once. An event can be processed repeatedly, but their effects on managed state are applied exactly once. If an event is replayed, than the state is restored to some point before the event, to maintain consitency.
