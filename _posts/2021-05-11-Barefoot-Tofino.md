---
title: "TCP/IP model (previously, OSI 7-Layer)"
date: 2021-05-12
categories:
  - Network
tags:
  - TCP/IP model
---

Before start, TCP/IP model has mostly replaced the tradinal internet communication model (OSI 7-Layer), and these are mostly similar.


# Basic Components

### bf_switchd: user-space driver
- To interface with the running P4 program 
- Load program-dependent APIs, and initialize Thrift server for them
- Thrift: SW framework to develop RPC handling among different languages
- Once the driver is started, we will see the bfshell prompt
- Configuration by a JSON file 
##### BF-Shell: Barefoot Interactive Shell
- *bfshell>* is the top-level
- we can access various subsystems from here
- shell outputs its logs on this prompt, and these logs will be intermixed with our work
- to avoid this, we can access the shell remotely 
- how? run *run_bfshell.sh* to access the CLI thread of the driver (if we run commands here, we can see the output logs in the driver window)
- more efficiently, we can ask *run_bfshell.sh* to run the setup script and move to bfrt_python



### bfrt_python
- A tool to control tables (and other P4 objects) in my program
- *bfrt_root* is the root object
- From this, we can access ongoing P4 programm and its objects

### bfrt_client vs bfrt_server
- bfrt_client: SDN-like remote controller implementation
- bfrt_server: replaced by Thrift for now (same role) i.e., take RPC from bfrt_client and handle


##### References
[1] https://www.monolithicpower.com/en/analog-vs-digital-signal
[2] Data Communications and Networking 5th edition (Behrouz Forouzan)
