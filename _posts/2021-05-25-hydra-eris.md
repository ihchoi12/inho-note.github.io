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
- Multi-stamp: set of sequencer numbers assigned to each groupcast msg

### Support General TXs
- Use independent TXs as a building block
- *preliminary TXs*: to gather read dependencies
- commit *preliminary TXs* with a single *conclusory independent TX*
- it imposes locking overheads, but still efficient by using the efficient underlying independent TX primitive.


# Implementation
[kvClient.cc]
#1: decide independent vs multi-phase TX 
#2: randomly select n ops: keys and their shards (say m shards) (wPer% PUT, rest GET)
[client.cc]
#1: separate ops by shard => m msgs (containing ops to the shard)
#2: serialize each msg to request => map(shard, request)



##### References
[1] https://www.youtube.com/watch?v=Vmb8xGD-LV8
