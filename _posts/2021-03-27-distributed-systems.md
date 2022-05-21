---
title: "Distributed Systems"
date: 2021-03-26
categories:
  - Distributed Systems
tags:
  - Test
---


# RPC vs msg passing
- RPC: similar to a function call, but the function is on a remote process
- So, RPC is blocking mechanism: call a remote function, and block (at least this RPC thread) until the function is completed
- Whereas, msg passing is non-blocking: send a message and continue to do sth else


# Distributed States
## Network File System
- Traditional FS locally manages files on a device
- NFS stores files on a remote server (e.g., Dropbox), so that the files can be accessed anywhere via network

### How it works?
- a client (other user machines) fetchs data from server everytime it wants to access
- two issues: high latency, increase load on the server
- impvore: client-side cache (fetch first time, and store it on the local cache)
- two benefits: low latency. reducd load on the server (exploiting **locality**)
- Introduction of Distributed States: multiple client machines with their own cache for each
- issue: a client want to write a file => write to it's local cache => other clients' state are old
- this issue is handled in different ways depending on the systems
* cache: to move data to where we want to use it
* RPC: to move computation to where the data is

### Example: NFS
- developed by Sun Microsystems in 1984
- design philosophy: simplicity
- principle 1: stateless
  * all essential information kept on server's disk
  * can still cache in server memory (to save the memory access latency), but cannot depend on cache for correctness
  * servers do not cache client information
- principle 2: idempotent operations
  * read and write at offset
  * lookup

##### NFS Update protocol
- case: client writes to a file
  * update local cache
  * send write RPC request to server (client's write-through cache: keep re-sending until the server replies)
  * server write data to disks, then reply back to the client
  * (once the client receives the reply, it knows that the data is persisten on the server)
- Issue1: write need to go through network, server need to access disk => slow
- Issue2: some write operations are unnecessary if they are overwritten by later operations 
- Issue3: other clients' cache still have old states
- NFS solution: 1) periodic polling => eventually receive updates (NFS just accepts stale data in between pollings)

### Example: Sprite
- Unix-like distributed OS from Berkeley
- Server tracks opend file state
  * which clients are reading/writing files
  * open()/close() needs to contact the server
- Server uses write-back cache
  * modified blocks are kept in memory
  * writes back to disk every 30s

##### Sprite consistency
- leverage that server knows which clients are reading/writing a file
- If only one client opens a file
  * client can manage the data on it's cache locally
  * no need to synchronously sends updates to the server
  * writes back to the server every 30s
- once server knows that multiple clients open a file and at least one is writing
  * send uncache(file) to all cleints who are cacheing the file (server knows them from it's client information)
  * clients reply back with the current cache of the file, and promise that they will not cache it anymore
  * all clients become non-cacheable for the file
  * from now on, all reads and writes go through the server (non-cacheable)
  * server writes the up-to-date file to the disk every 30s
- since server maintains the client information in the memory, recovery from server failure requires reconstruction of the client information for consistency
- Sprite tradeoff between advantage (consistency, performance) and disadvantages (complexity, durability & recovery cost) 


# State Machine Replication (SMR)
Goal: agree on order of ops (i.e., sequential log of ops) \
That is, different servers propose different ops for the same slot, so need consensus for their order

### Primary-Backup Limitations
- Split-brain: cannot know if a primary has failed or is just slow => if it's just slow, might end up with two primaries
- View Server can prevent this issue. But it's a SFP.
- Replicate View Server using Primary-Backup? Same issue again..
- Q. How can we achieve SMR without relying on a single entity? A. Distributed Consensus

### FLP: prove impossibility of a deterministic protocol to guarantee consensus in bounded time in asynchronous DS (even if only one server can fail)
- Then, what shoud we do?
- We still can achievethe distributed consensus with some assumptions

### Paxos: 
- Goal: consensus on a single value (note that it's not an order of ops by just one round of Paxos)
- Under two assumptions: 1) majority are alive; 2) no failure for a sufficiently long period;
- Important point!: a value is chosen only once a proposer with the value has been accepted by a majority. In other words, if a value has been accepted by a majority with different proposal-numbers, it has not been chosen.


##### Three types of players
- Proposers: send (uniquely numbered) proposals with a value to be chosen (accepted by a majority)
  * "accepted" is a local state, "chosen" is a global state of a 'proposal' (not a value), so a learner need to contact multiple acceptors to learn a chosen value
- Acceptors: accept/reject proposals, collectively decide when a value is chosen
- Learners: once a value is chosen, learn it
- All these are logical roles (multiple roles can co-locate on the same physical node)

##### Two rules
1. Safety
  * only a single proposed value can be chosen (that is, if a value is chosen, the event of chosen can happen only for the same value with a higher psn)
  * only chosen values can be learned
2. Liveness
  * as long as there are proposals, there is a chosen one, and a process eventually learn it


##### Hwo it works?
- A proposer broadcast (prepare, psn: N)
- Acceptors who receive (prepare, psn) promise to reject psn < N, and reply their (accepted, psn, val)
- The proposer wait for the replies from a majority acceptors, and choose val in (accepted, psn, val) with the max psn, and send (accept, psn, val) to the majority acceptors who replied (Q. why explicitly only to the replied acceptors? can it be broadcast? no broadcast since it breaks safety) 
- Acceptors who receive (accept, psn, val) and hasn't promised to reject psn accept it, and send (accepted, psn, val) to distinguished learner (DL)
- If DL receives (accepted, psn, val) from a majority of acceptors, val is learned and broadcast (decided, val) to all learners 
- If DL timeouts without learning a chosen value, it requests proposers to propose something again

##### Distinguished Proposer (DP)
- Why? Paxos does not guarantee liveness in the case that proposers compete each other recursively (promise 1 => promise 2 => reject 1 => promise 3 => reject 2 ..)
- Of course, election of DP itself is a consensus, so it's not guaranteed that all nodes agree on a single DP (multiple DP can exist), but it does not break the safety since its only for liveness (to make the Paxos more likely to reach the consensus eventually)


### Then, how can we extend it to SMR (sequence of ops)? PMMC
- A leader for all slots is elected by Paxos phase 1
- Once elected, the leader runs only phase 2 for each proposal
- If leader dies, other nodes repeat the process
- Paxos ensure safety

##### Roles in PMMC
- Replicas (like learners): maintain the log of ops, state machine (apply decided ops) 
- Leaders (like proposers): try to elect themselves, drive consensus protocol
- Acceptors (like acceptors): vote for leaders, accept ballots: (seq-num).(leader-id)

##### 
- client sends request (A) to the replicas 
- a replica broadcast proposal(idx, A) to all leaders
- a leader receives proposal(idx, A), insert it in the proposals \[(idx, A)\], (if is_active), send p2a
- if the leader knows that it's proposal is chosen (receive matching p2b from majority), broadcast the chosen (idx, val) to all replicas 

##### Leader Election
- When? 1) At the beginning of time; 2) When the current leader seems to fail (haven't header heartbeat msgs) 
- Local states: is_active(false), ballot_num (0.0), proposals \[\] 
- The leader broadcast p1a(0.0) to acceptors
- Some acceptors reply p1b(0.1) and the leader fails to get mojority p1b(0.0, \[\]), so preempted (if a leader gets preeempted, don't try immediately again!)
- The leader increase it's ballot_num to 1.0 (to be bigger than 0.1), and (if the current leader seems to fail) broadcast p1a(1.0)
- The leader gets a majority p1b(1.0, \[\]), set is_active(true), put the accepted (idx, val) in p1b msgs with the highest ballot_num in proposals, then can start to send p2a msg  

##### Acceptor's logic
- leader 1 broadcast-with-retry p1a(ballot_num: 0.1) to acceptors 
- An acceptor receives p1a(0.1), compare 0.1 to its current ballot_num (null now), set ballot_num: 0.1 (promise), and reply p1b(0.1, accepted: [])   
- leader 1 receives p1b(0.1, []) from majority acceptors, then send-with-retry p2a(0.1, idx: 0, val: A)
- An acceptore receives p2a(0.1, 0, A), compare ballot_num, accepte it ONLY IF THEY ARE EQUAL, set accepted \[<0.1, 0, A>\], reply p2b(0.1)


### Raft
##### Issues in Paxos (or PMMC)
- hard to understand
- multi-paxos protocol not well specified
- Separate roles: proposer, acceptor, learner, which doesn't match system deployment in practice 

##### Idea
- Only one role: replica (or server)
- 3 states: Leader, Follower, Candidate (a server can only be in one state at a time)
- Based on an operation log
- 2 main protocols: 1) leader election; 2) log replication;

##### Leader Election
- Leader: similar to the distinguished leader in Paxos and active leader in PMMC
  * Only leader handles client requests, and replicates them to followers
  * sends periodic heartbeat msg to followers
- Term: Divides time into terms: consecutive integers e.g., term 1, 2, 3, ...
  * each term begins with an election
  * at most one (or no) replica will win the election in a term
  * each replica remember the latest term number observed, and ignore msg with lower term number
- How it works
  * all replicas begin as followers
  * if a follower receives no msgs from the leader over timeout, start election
  * the follower: term++, transition to candidate state, vote for itself, broadcast RequestVoteRPC
  * a replica can only vote for at most one candidate in a term (FCFS)
  * a candidate wins if receives majority votes, then transition to leader state
  * votes can be split w/o forming a majority, then candidate will timeout ant start a new election again
##### Log Replication
- the leader receives a request
- append the request to its log
- broadcast AppendEntriesRPC with idx and term of the immediately preceding the new entry (retry until all followers respond)
- a follower rejects the AppendEntries request if it doesn't have the preceding entry, otherwise append the new entry and reply to the leader (implication: no holes in the log)
- after the entry is stored on majority, execute request and reply

##### Log Commit
- once the leader replicates an entry on a majority replicas (similart to 'chosen' in Paxos), the entry is committed
- the latest committed entry is the log commit point
- implies that all prior entries are also commited
- so, safe to execute commands to the log commit point

##### Handling Log Divergence
- due to the network anomalies, a replica may have different log entries than other replicas
- once a follower receives AppendEntriesRPC from the leader, it checks if the preceding entry is matching, if not delete it and reject the AppendEntriesRPC
- then, leader retries AppendEntriesRPC with a lower indexed entry
- repeat it until the follower finds a matching entry, then the leader sends all following entries to the follower (i.e., log duplication)
- it means that the leader can modify other replicas' log, so we need to guarantee that a new leader has all committed entries so far (otherwise, commited entries can be overwritten by the leader), but how? 
- Use majority intersection: a replica rejects RequestVoteRPC if it has more up-to-date (i.e., higher term number, longer log) log than the sender

## SMR in real systems
### Chubby
- Distributed coordination service
- Extensively used in Google
- Apache Zookeeper: equivalent open-source version of Chubby 
##### Chubby's goal
- allow to synchronize and manage dynamice configuration state
- fault-tolerant, available, and strongly consistent (linearizable)
- intuition: only some parts of an app need consensus!
  * dslabs-lab2: view service
  * master/leader election in DFS
  * Synchronization in multi-threaded programs
- implementation: multi-Paxos SMR

##### History
- Google has many services that need reliable coordination
- originally they did ad-hoc things (e.g., implement Paxos), which is very hard and prone to error
- Let's build this service available to apps! 

* Question: does locking mechanism requires linearizability to be correct? Cannot be sequential consistency?
* Answer: Only linearizability. Why? The key difference between linearizability and sequential consistency: whether stale-read is allowed. If stale-read is allowed, the lock-server may falsely respond that no one is holding the lock, which is wrong.

##### Interface
- like a file system which provides:
  * (small) files to record the metadata (e.g., who is the leader, etc.)
  * locking
  * sequencers: to assign seq_num to the locks to deal with the asynchrony (i.e., to discriminate if a request to a file is with an old lock)
- API:
  * Open, Close
  * GetContents, SetContents, Delete
  * Acquire, TryAcquire, Release (for locks)
  * GetSequencer, SetSequencer, CheckSequencer

##### Performance Optimization
- so far, it's almost same as the multi-Paxos, which has all required guarantees
- but the issue is performance
- need to engineer it so that we don't need to run Paxos on every RPC
- Idea: batching (for throughput), partitioning, non-replicated read requests, client caching

##### lease for non-replicated read requests optimization
- read requests need to be replicated only to guarantee that stale-read doesn't happen
- stale-read happens when a replica falsely think that it's the leader and serve the read
- but if we can guarantee that the read is executed by the actual leader, we don't actually need to replicate it
- so we use lease: a new master gets lease and renew it while it is up, and only a master with valid lease can perform the read w/o replication
- the lease doesn't handle the clock skew issue completely, so it doesn't 100% guarantee linearizability (but the probability of violation is very low with time synchronization protocols and conservative non-leaders who wait a bit longer than the timeout of the lease so that a new leader is not elected before the lease on the leader is actually expired)

##### client caching
- to reduce the number of clients' RPCs to Chubby
- once a client reads a file, cache the file data and metadata
- for linearizability, write-through, write-invalidate cache
  * Master tracks which clients might have file cached
  * Sends invalidations on update


### Google File System (GFS)
- Google needed a DFS to store search index
- why not use existing FS (e.g., NFS, AFS, etc.)?
  * very differnet workload chateristics
  * Require strong fault tolerance because components fail constantly 
- Require high throughput and bandwidth (latency is a secondary concern)

##### Workload
- most of files are very large (multi-GB) => no need to be optimized for small files
- reads: small random reads and large streaming reads
- writes: 
  * many files written once (i.e., immutable), other files appended to
  * so, small random writes are very rare
##### interface
- app-level library, not a POSIX file system like NFS
- function calls: create, delete, open, close, read, write
  * concurrent writes are not guaranteed to be consistent
- record append operation: guaranteed to be atomic and consistent
- snapshot operation

##### Architecture
- each file stored as 64MB chunks
- each chunk on 3+ chunkservers
- a single (replicated) GFS master stores metadata only







# Transaction Concept
Goal: to group a set of operations into an **atomic** unit \
4 guarantees: ACID \
What is **Distributed** TX? TX on a **partitioned** (into multiple shards) DB (while preserving ACID) \
Why partition? 1) storage capacity; 2) scalability (by distributing client requests); 

### Atomicity
- Executed (committed) by all participating shards or none (aborted) 
- by **Two Phase Commit** (two rolse: 1 Coordinator & multiple Participants) \
a) C sends PREPARE (for a TX) to Ps \
b) P sends 1) YES (and waits for the decision); OR 2) NO (and unilaterally abort) \
c) If all Ps vote YES, C sends COMMIT to Ps \
d) Otherwise, C sends ABORT to those voted YES \
* The Coordinator decides the order of TXs consistently across shards (like the leader in Paxos)

### Isolation
- Isolation of TX is similar to Consistency of distributed systems. (Consistency in ACID means consitency of DB)

\[Consistency of OPs in DS\]
- Sequential Consistency (also called Serializability, especially for TXs): 
history of operations is equivalent to a sequencial order **respecing local ordering**. \
- Linearizability: Sequential Consistency + repects real-time ordering. \
\[Isolation of TXs in DB\] \
- Serializability: execution results of TXs are equivalent with running them in a serial order, one at a time \
- Strict-Serializability: Serializability + repects real-time ordering. \
(17:26)

##### Two-Phase Locking
- Two types of locks: 1) read-lock (shared -- multiple holders can exist); 2) write-lock (exclusive -- at most one holder always, if someone holds it for an object, no one can hold even read-lock for the object)
- Why two types? Having just one exclusive lock is correct. However, we want to have the maximum concurrency (ex. read-only TXs can be executed concurrently).
- Phase 1 (Expanding Phase): only acquire locks
- Decision made (TX commit or abort)
- Phase 2 (Shrinking Phase): only release locks
- Why TPL guarantee **serializability**? exclusiveness of write-lock and two-phased approach prevent conflicts of TXs
- Why TPL guarantee **strict serializability**? strict means: if TX B comes in after A commits (in real-time), B is ordered after A. In this case, expanding phase for A will be started first, so B cannot be ordered before A


### Durability
- TX effect persist after commit
- single node DB case: use write-ahead logging and persistent storage
- what if disk fails together?
- replicate DB using Paxos
- distributed DB: replicate each shard

# Spanner



# Byzantine Fault Tolerance
### Byzantine Fault
- faulty nodes can take arbitrary actions

### Simple example: can we tolerate 1 BF with 3 (2f+1) nodes?
- No
- Assume 3 nodes: A, B, C (C is BF)
- Initially 'X = 0', a client request 'X = 1'
- A and C agree on 'X = 1', client get reply
- client send get(X) to B and C, C lies that 'X = 0', client get 'X = 0' 

### BFT SMR
- assume f replicas are Byzantine faulty
- still want to guarantee linearizability
- how many nodes do we need?
  * let's say we have n nodes
  * since there are f faulty nodes, the system must be able to reach consensus with at least n-f nodes
  * but actually faulty nodes could be in those n-f nodes
  * so there are at least n-2f benign nodes among those n-f nodes, and it should be bigger than f
  * n-2f > f ==> n > 3f
- What is the quorum here?
  * majority is not enough (why? majority intersection is at least one, but what if the one is byzantine faulty?)
  * so we need at least f+1 intersection (i.e., at least one in the intersection is benign)
  * so, quorum is 2f+1

### PBFT
- progress through a series of views
- a single primary associated with each view
- the primary assigns the command a slot-num and forwards to backups

##### How to handle byzantine faults (Idea)
- Case: primary sends a wrong result to the client
  * all replicas send replies directly to the client (iff the command is committed)
  * clients wait for f+1 matching replies
  * then, at least 1 reply is non-faulty (i.e., the mathicng reply is basically the non-faulty), and it is committed, so it's correct reply
- Case: primary assigns different commands to the same slot-num (i.e., equivocation)
  * we cannot rely only on the primary's msg
  * so, replicas exchange information received from primary (to make sure primary is not equivocating)
- Case: primary ignores clients, or any other ways to prevent progress
  * backups do a view-change protocol when timing out committing client requests
- Case: primary or backups impersonate other nodes
  * sign all msgs using cryptography (public&private key)

##### Protocol Overview (3 sub-protocols)
- 1. Normal operations
  * phase1: pre-prepare
  * phase2: prepare
  * phase3: commit
- 2. View change
- 3. Garbage Collection

##### Phase0: Request
- client sends request (m) to the primary
- we assume that clients are non-faulty here

##### Phase1: Pre-prepare
- Why? to broadcast some information (view, seq_num, request) to all replicas
- The primary broadcasts PRE-PREPARE with followings
  * view
  * seq_num
  * hash(request msg) // sign upto this point
  * request_msg
- PRE-PREPARE is signed by the primary
  * optimization: when signing, use hash of request msg and send original msg separately
  * then, backups can verify request msg using the hash operation
  * also, significanty reduce the sigining cost 
  
##### Phase2: Prepare
- Why? a BF primary can do equivocation (sending different requests for a single slot), so backups need to ensure that they've received the same thing
- Each backup broadcasts PREPARE with followings
  * view
  * seq_num
  * hash(request msg) // sign upto this point
- If a backup receives 2f matching PREPAREs: 
  * it's guaranteed that it's the ONLY request that can be prepared for the view and seq_num
  * why 2f not 2f+1? one from primary received already by PRE-PREPARE
  * It's called that the (view, seq_num) has the Prepare Certificate (i.e., no other perpare certificate with other requests exist)


##### Phase3: Commit
- Why? Perpare Certificate guarantees that no other requests can be perpared for the same (view, seq_num), howerver it deosn't guarantee for (view', seq_num). To prevent this, we need to ensure that there are enough replicas with the same Prepare Certificate.
- 


##### Phase4: Reply
- Clients wait for f+1 matching replies
- Then, at least one reply is from correct node


Q. what if a faulty primary broadcast a wrong request (m) to backups?
A. it cannot happen because m is signed by the client

### Useful Commands for Hydra

$ lscpu | grep NUMA
// to check NUMA node and CPU core mappings 




