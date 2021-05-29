---
title: "Hyera-Eris"
date: 2021-05-25
categories:
  - Research
tags:
  - Hydra
---
# Eris Paper
- two responsibility: ordering TXs, atomicity
- traditionally, Paxos for replication, TPC for atomicity, TPL for isolation
- Eris: no coordination for independent TXs, minimal overheads for general TXs
- multi-sequencing: core network primitive

### Three Design Principles
- Separate ordering (remove TPL during execution)
- Use in-network ordering (multi-sequencing)
- Integrate intra- and inter- shard coordination

### TX: Independent vs General 
- Independent: atomic execution of a single TX across multiple shards
- No cross-shard dependencies
- Liniarizability by just sequentially executing in a consistent order
- read-only TX can be expressed as an independent TX
\
- General:


### In-network Concurrency Control
- Groupcast: deliver msg to set of multicast groups
- Use special IP addr which is specially processed 
- Additional header: a list of destination groups (between IP and UDP)
- DL replica: leader of it's shard
\
- Multi-stamp: set of <group-id, seq-num> assigned to each groupcast msg
- In case of packet drop, receiver can request the missing pkt by the seq-num even for other group-ids

### FC: Failure Coordinator
- handle pkt drops and sequencer failures (through interaction with replicas)

### Fault Tolerance and Epoch
- Epoch-num: to handle sequencer failure (maintained by the sequencer), added to the GC header
- Controller handle sequencer failover (sequencer timeout => selecte new sequencer => epoch-num++)
- Receive see higher epoch-num => do view-change

### 5 Protocols of Eris
1) normal case: txn-index (i.e., opnum), cannot process TXs in *perm-drops* (to be NO-OP) or *temp-drops* (candidates to be NO-OP)
2) handling pkt drops: FC ensures atomicity 
3) DL change (within a shard)
4) Epoch change
5) Synchronization

### Handling Pkt Drops
- A replica detect a pkt drop => send <FIND-TXN, txn-id>, txn-id: <shard-num, seq-num, epoch-num> to FC
- FC broadcast <TXN-REQUEST, txn-id> to all shards all replicas
- If a receiver has matching TX, replies <HAS-TXN, txn>. Else, add txn-id to *temp-drops* and replies <TEMP-DROPPED-TXN, view-num, txn-id>
- FC receives 1) a single HAS-TXN => save it and sends <TXN-FOUND, txn> to all; 2) quorum TEMP-DROPPED-TXN from every shard => send <TXN-DROPPED,
txn-id> to all replicas;
- Replica receives 1) TXN-FOUND => add txn into *un-drops* and log, replies back to client; 2) TXN-DROPPED => add txn-id into *perm-drops*, add NO-OP to log
- Replica proceeds to subsequent TXs
* OPTIMIZATION: pkt drop detect => tries to recover from other replicas within its shard first (can handle this drop without FC)



### Support General TXs
- Use independent TXs as a building block
- *preliminary TXs*: to gather read dependencies
- commit *preliminary TXs* with a single *conclusory independent TX*
- it imposes locking overheads, but still efficient by using the efficient underlying independent TX primitive.

# Implementation
### SDN Controller
- POX
- manages GC membership
- install routing rules (GC pkts through the seqeuncer)
### End-host lib
- API for sending and receiving multi-sequencer GC pkts
- Monitors incoming multi-stamp (to send DROP of NEW-EPOCH notifications)
### P4 Sequencer
- How many num_shards can be suported? RMT: 32 stages, 4-6 registers per stage => 128-192 destinations per pkt
- Pkt header bytes? 512-byte => assuming 32-bit group-id and seq-num, 116 destinations
- Beyond that? need special-case handling





# Codebase
[kvClient.cc]
#1: decide independent vs multi-phase TX 
#2: randomly select n ops: keys and their shards (say m shards) (wPer% PUT, rest GET)
[client.cc]
#1: separate ops by shard => m msgs (containing ops to the shard)
#2: serialize each msg to request => map(shard, request)



##### References
[1] https://www.youtube.com/watch?v=Vmb8xGD-LV8
