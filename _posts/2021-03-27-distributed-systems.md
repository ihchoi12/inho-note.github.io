---
title: "Distributed Systems"
date: 2021-03-26
categories:
  - Distributed Systems
tags:
  - Test
---

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

[Consistency of OPs in DS]
- Sequential Consistency (also called Serializability, especially for TXs): 
history of operations is equivalent to a sequencial order **respecing local ordering**. \
- Linearizability: Sequential Consistency + repects real-time ordering. \
[Isolation of TXs in DB] \
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







### Useful Commands for Hydra

$ lscpu | grep NUMA
// to check NUMA node and CPU core mappings 




