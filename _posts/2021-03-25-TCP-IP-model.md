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


# Layer-2: Data-Link Layer 
### How multiple nodes communicate?
- Just connect all pairs among them by cables? Not scalable...
- **Switch**: Okay, let's create a shared machine, and connect all nodes to it (i.e., form a network). 
- Then, what if two nodes are in different networks? 
- Connect the switches (at this point, the switch becomes a **router**) 
- In this way, we connect all computers in the world (Internet)

### Okay, multiple nodes in a network are connected through a switch. Then, how to specify the destination among them?
- MAC address: gloablly unique address of each network-compatible device.
- L2 Switch use MAC address to decide the destination and forward it.  

### Assume a node receives a long stream of signals from multiple senders. How to separate them?
- Framing: sender appends a certain bitstream before and after the data to be sent

##### Data-Link Layer: modules (implemented in a hardware called LAN card) enabling framing and de-framing.



# Layer-3: Network Layer 
### Assume node A and B are in different networks. How A specify the destination B for data to send
- A writes IP address of B (get IP using DNS) into the IP header. Now, this messages is called **packet**.

### Once receiving a packet, how switch (or router) know where to forward the packet?
- Parse the destinaion IP address from the packet's IP header, and use it to decide next path (routing)
- Write the IP header again and forward

##### Network Layer: Modules (implemented in OS kernel software) enabling destinaion addressing (by IP) and routing (deciding next path using IP address) & forwarding on inter-network environement. 

### Now, all computers in the world can communicate each other using Layer 1-3.

# Layer-4: Transport Layer 
### Assume node A has received multiple packets. How A know to which process it should give each packet?
- **Port Number**: let's differentiate processes using locally unique numbers
- Sender write the port number into the L4 header.

##### Transport Layer: Modules (implemented in OS kernel software) handling port number of packets to specify the final destination (i.e., a specific process). 

# Layer-5: Application Layer
### Assume a process receive a packet. How to decide specifically what to do with the data?
- Write some additioinal information with the data
- Use those information for the specific processing in the application layer

### TCP/IP socket programming
- Developing a communication-enabled programm using APIs in OS's Transport Layer (Layer-4).
- Using it, anyone can implement customized applycation layer. 

##### Application Layer: Modules (implemented in a program) encoding or decoding some specific information for the specific application-level processing. 

##### References
[1] https://www.monolithicpower.com/en/analog-vs-digital-signal
[2] Data Communications and Networking 5th edition (Behrouz Forouzan)
