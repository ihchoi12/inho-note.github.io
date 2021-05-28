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

# Implementation
[kvClient.cc]
#1: decide independent vs multi-phase TX 
#2: randomly select n ops: keys and their shards (say m shards) (wPer% PUT, rest GET)
[client.cc]
#1: separate ops by shard => m msgs (containing ops to the shard)
#2: serialize each msg to request => map(shard, request)

### Independent TX
- atomic execution of a single TX across multiple shards


##### References
[1] https://www.youtube.com/watch?v=Vmb8xGD-LV8
