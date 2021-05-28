---
title: "Distributed Systems"
date: 2021-03-26
categories:
  - Distributed Systems
tags:
  - Test
---

###### Transaction Concept
Goal: to group a set of operations into an **atomic** unit \
4 guarantees: ACID \
What is **Distributed** TX? TX on a **partitioned** (into multiple shards) DB (while preserving ACID) \
Why partition? 1) storage capacity; 2) scalability (by distributing client requests); 

###### Atomicity
- Executed (committed) by all participating shards or none (aborted) 
- by **Two Phase Commit** (two rolse: 1 Coordinator & multiple Participants) \
a) C sends PREPARE (for a TX) to Ps \
b) P sends 1) YES (and waits for the decision); OR 2) NO (and unilaterally abort) \
c) If all Ps vote YES, C sends COMMIT to Ps \
d) Otherwise, C sends ABORT to those voted YES \

###### Isolation
- Isolation of TX is similar to Consistency of distributed systems. (Consistency in ACID means consitency of DB)

[Consistency of OPs in DS]
- Sequential Consistency (also called Serializability, especially for TXs): 
history of operations is equivalent to a sequencial order **respecing local ordering**. \
- Linearizability: Sequential Consistency + repects real-time ordering. \
[Isolation of TXs in DB] \
- Serializability:  \
- Strict-Serializability: \
(17:26)


Useful Commands for Hydra

$ lscpu | grep NUMA
// to check NUMA node and CPU core mappings 




