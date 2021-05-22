---
title: "TCP/IP model (previously, OSI 7-Layer)"
date: 2021-05-12
categories:
  - Network
tags:
  - TCP/IP model
---

Before start, TCP/IP model has mostly replaced the tradinal internet communication model (OSI 7-Layer), and these are mostly similar.


# Layer-1: Physical Layer 

### How two nodes (computers) communicate?
- ALL programs and files are sequences of 0 or 1, so the only thing they need is a way to send and receive 0 or 1
- Looks simple! Why don't we connect them via a electric cable, and assume 0V is 0 and 5V is 1 (i.e., digital signal) [1]?
- It doesn't work.. why? such digital signal have 0 ~ infinite frequency, and no cables can transmit such signal 
- Then, how can we send digital signal? Solution: encoding it into analogue signal\

##### Physical Layer: modules (implemented in a hardware called PHY-chip) enabling two nodes' communication by 1) Encoding (digital => analouge); 2) Upon receiving analogue, decoding (analouge => digital);


### bf_switchd: user-space driver
- To interface with the running P4 program 
- Load program-dependent APIs, and initialize Thrift server for them
- Thrift: SW framework to develop RPC handling among different languages
- Once the driver is started, we will see the bfshell prompt 
##### BF-Shell: Barefoot Interactive Shell
- *bfshell>* is the top-level
- we can access various subsystems from here
- shell outputs its logs on this prompt, and these logs will be intermixed with our work
- to avoid this, we can access the shell remotely 
- how? run *run_bfshell.sh* to access the CLI thread of the driver (if we run commands here, we can see the output logs in the driver window)
- more efficiently, we can ask *run_bfshell.sh* to run the setup script and move to bfrt_python


### bfrt_python
- to control tables (and other P4 objects) in my program





##### References
[1] https://www.monolithicpower.com/en/analog-vs-digital-signal
[2] Data Communications and Networking 5th edition (Behrouz Forouzan)
