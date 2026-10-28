# Blockchain

Byzantine problem:
    - Group of members must decide something
    - Can message each other, but messages are unreliable
    - Message can also be from traitors
    - Loyal generals must reach consensus


## Consenus problem
Given:
- variable $x$,
- $n$ nodes,
    - each node $i$ has initial value $X_i$ for $x$
    - subset of $f$ faulty nodes and $n-f$ correct nodes
- nodes communicate by sending messages

Have the correct nodes agree on a common value $X$ for $x$.

Crash failure is a possible fault model, many other possible

> Crash failure: A node executes correctly up to some point when it stops.

Concseus Protocol must satisfy:
1. __Agreement__ If two correct nodes decide, they decide on the same value.
2. __Validity__ If a correct nodes decides on $X$, then $X$ was proposed by some node
3. __Termination__ All correct nodes eventually decide on a value


### Account
A private public key pair, account is identified by the public key.

Only the private key holder can sign transactions from the account.

No central authority distributing accounts

### Transaction
Transfer or value from input to output accounts.
transaction fee $\Delta = \sum \text{inputs} - \sum \text{outputs}$

A transaction from A to B contains: public key of A, and a signature of the previous transaction of A



### Ledger
Transactions are combined into blocks. blcoks are appended in sequence.
They ahve a heaer of 80 bytes with timestamp, and a backhash (of the previous block) and other metadata

Payload of up to 4MB with transactions

### Mining
Client nodes: create transactions and broadcast through the network
Miner nodes:
- collect transactions to form a new block
- Solve a puzzle (hash with 0 prefixes)
- Broadcast the new block with proof of work

Full nodes: collect mind blocks and verify them

the proof of work is: find a nonce s.t. hash(block+nonce) <= a globally known 256 bit integer
Only brute force is posible
Nonce is called the proof of work.

The miner puts a Coinbase trnasaction as the first tranascction of the block. It contains the blocks current subsidy, and the sum of the transaction fees, transacted to the public key of the miner. It can choose which transactions are included, so it likely chooses the transactions with the highest fees.

Goals:
1. Limit communication overhaed to O(n) -> a new block is communciated once to everyone, initaited by the miner
2. Maintain integrity of the blockchain


### Verification
- Nonce verification
- Backhash verification
- Transactions: double spending, output >0 and inputs <= outputs
- coinbase transaction


### Resolving forks
miners append block to the local blockchain. Chain may fork when blocks are solves at similar times.

Solution:
- Append block to longest path, if they are of equal lengths, work on the block first received. 
- Adapt PoW dfficulty to average 10 minutes block creation time.
The difficulty is decided by the bitcoin software. It is the same for everyone, so if one tries to cheat, other can see it.


### Incentive
Goal is to incentivize maintaining the blockchain, and defer attackers

A miner is rewarded BTC for sovled blcok (3.125 currenty), plus transaction fees.

Only paid if block is valid, trivially.

Only the reward in the global chain counts, i.e. the longest chain. requires a 51% attack to have invalid blocks accepted.


##  Properties
- Probabilisit consensus guarantee
- Permissionless, zero trust needed
- Scalable to large networks, only O(n) messages sent per new block

## issues:
- PoW requires lots of energy, its costly
- Databse replication is costly, all miners and full nodes must keep a copy of the whole blockchain
- Large latency, bitcoinc 10 minutes between blocks, potentaily have to wait long for a transaction verification especially with low fee
- Low throughput, due to 10 minute per 4MB transaction space.

## Solutions
- PoW replace by PoS: e.g. euthereum. You need to invest your own money and put it at stake, in order to be able to vote. If oyu vote incorrectly, you lose moeny. If you vote honestly, you gain some.
- Database sharding (partitioning) instead of replication
- PAyment channels: move small transaction off-chain and only put results there.
