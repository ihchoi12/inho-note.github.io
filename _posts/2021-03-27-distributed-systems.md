---
title: "Distributed Systems"
date: 2021-03-26
categories:
  - Distributed Systems
tags:
  - Test
---

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

Useful Commands for Hydra

$ lscpu | grep NUMA
// to check NUMA node and CPU core mappings 




