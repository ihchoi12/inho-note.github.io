---
title: "IRQ"
date: 2021-05-12
categories:
  - OS
tags:
  - 
---

# PCIe slots
- Peripheral Component Interconnect Express slots
- Used to connect peripheral devices (e.g., graphics cards, expansion cards, USB adapters) to the computer's motherboard 
- Several types depending on the number of data links - called PCIe lanes: PCIe x16 has 16 lanes

### PCIe versions
- It's assigned to both sides: slots and peripheral devices
- 2004: PCIe 1.0 - 250MB/s per lane
- 2007: PCIe 2.0 - 500MB/s per lane
- 2010: PCIe 3.0 - 985MB/s per lane
- 2017: PCIe 4.0 - 1.97GB/s per lane


### How PCIe handles data?
- Data is communicated to and from PCIe slots via PCIe lanes 
- Given the data speed on a single lane is fixed, having more lanes means more data is communicated per unit time (faster)
- Usually the PCIe slot with more lane is physically bigger 

### High compatibility of versions and slots' size
- PCIe 3.0 grphics card connected to PCIe 2.0 slot => works find with PCIe 2.0 speed
- An expansion card requiring 4 lanes can be connected to a PCIe x16 slot
- A device with x16 PCIe card can be connected to a PCIe x4 slot with x16 size, or open-ended slots (the speed will be PCIe x4)

# NIC
- a device (a piece of peripherals) to enable a computer to communicate with other devices in the network
- connects to a system bus (mostly attched to a PCIe bus)
- multiple types of NICs: Ethernet, Wireless, InfiniBand (similar to ethernet but used in HPC)
- getting faster (100 GbE alreday) and more functional
- ASIC

### NIC Driver
- a programmed set of instructions and information requird by NIC to operate
- usually written in C/C++

### Some Example Functions of NICs
- RX/TX checksum offload: 
  * when we construct or receive a TCP or UDP packet, we need to calculate the checksum in the header (for verification)
  * it's mostly done by Linux software
  * NIC can take this task (to save CPU cycles)
- TCP/UDP segmentation offload
  * if we try to send a data bigger than MTU (usually 1500 bytes), it needs to be segmented into multiple packets
  * thie segmentation usually done by the Linux network stack software (which requires CPU cycles to segement data and calculate header for each segment)
  * With NIC, we can just give the entire data to NIC, and NIC handles the segmentation
- Large receive offload
  * reverse of the segmentation
  * when NIC receives multiple packets of a single stream, the NIC can aggregate them into a single big buffer, then send it to the kernel (saving CPU cycles)

# Network Processing in Linux
- NIC driver pre-allocates a ring-buffer in the main memory for the NIC
- NIC receives a packet
- NIC DMA copies the packet to a ring-buffer in main memory, then sends a interrupt to one of CPU cores
- The CPU core will go into kernel-mode, and run the kernel interrupt handler called IRQ handler
- The IRQ handler takes the packet from the ring-buffer, then performs packet processing (ETH, IP, TCP/UDP, ...), and finally get the actual payload, copies it to the memory provided by an user-space program (with the matching port number)
- Ex. a user-space program call recv(buf, ...), then the payload is copied to the buf

### A Potential Problem
- Linux kernel's network stack is pretty heavy
- When we have a high-speed network,
- The packet processing in kernel-mode IRQ handler becomes a throughput bottleneck or latency barrier (imagine, NIC receives 1pkt/hundreds-ns VS kernel processing 1pkt/a-few-us)
- So, processing all packets in the kernel is not a scalable solution
- alternative is kernel-bypassing

# Kernel-bypassed Networking
### DPDK: a software framework
- We link (or build) the dpdk libraries into the user program
- The DPDK libraries include PMD (poll-mode-driver: a user space driver allowing the application to talk to NIC in user space w/o going through the kernel)
- Then, the user-space program can directly poll the NIC
- Once the NIC receives a pkt, the program directly gets the packet and does the packet processing 
- Therefore, kernel is not involved, no context switching (to kernel mode) cost, no complex kernel mode network processing
- Issue1: safety because everyting is done in user-space (originally kernel provides security, rate-limiting, isolation, ...)
- Issue2: once DPDK binds to the NIC, that particular user program has the full control over the NIC (i.e., other programs cannot use the NIC anymore)

### Kernel-bypassing with OS Protection
- Key idea: separation of control plan and data plane
- Control plane: setup policies (protection, rate limiting, sharing, etc)
- Data plane: fast packet processing and forwarding

# Read/Write Remote Server's Memory
### Why Do We Want That?
- common in supercomputing/HPC
- increasingly popular in data center applications (data often doesn't fit in a single machine)

### Kernel-bypassing not Enough for This?
- The remote server's NIC receives a request pkt
- The pkt is directly processed by a user program in the server 
- The user program accesses the DRAM, and executes the request
- Here, the user program use CPU to process packet (higher performance overhead, waste CPU cycle)
- But! NICs can read/write local memory w/o CPU using DMA, so can we do it for remote memory as well?

### RDMA
##### Hige-level Idea
- A (RDMA enabled) NIC receives a request pkt 
- It bypasses CPU entirely (both kernel and user program)
- Instead, directly access the local DRAM (remote memory for the request sender), executes the request, then send back the result
##### How does NIC RDMA actually processes the pkts?
Two types of RDMA request primitives
- steps to use 1-sided primitives (READ, WRITE) in a RDMA program
  * Register a memory region (RM: registered memory) to allow the RDMA on each of client and server node
  * Pin RM into the physical memory (so that the RM is not swapped back to the disk)
  * Once its done, RDMA on the server side will return a remote-key (rkey) which is a permission to access the RM
  * The client receives rkey and address of RM from the server (now, RDMA connection setup is done!)
  * Then, client generates 1) queue pair (send queue (SQ) and receive queue (RQ) -- receive queue is used for 2-sided RDMA only; 2) completion queue (CQ);
  * Client RDMA application inserts the request (including the RM address and rkey) into the send queue, then starts to keep polling the CQ
  * Once the NIC has a free cycle, it picks up the request in send queue and send it to the remote NIC
  * The server NIC receives the request, verifies the rkey, DMA to the RM address, performs the request, get the result, and send it back to the client NIC
  * Client NIC receives the result, writes it to the local RM, then puts the completion event in the CQ
  * Client RDMA application sees the matching completion event in CQ, and learns that the requested data is in the local RM
  * To sum, client uses CPU (to put the request in the SQ), but server bypasses CPU and kernel entirely
- steps to use 2-sided primitives (SEND, RECV) in a RDMA program
  * no connection is needed
  * receiver side insert RECV request in RQ
  * once the receiver NIC receives corresponding SEND request, take the RECV in RQ and write the data in SEND to the RM
  * receiver also monitors the CQ (that is, it involves remote CPU as well, but not the kernel)
  * similar to DPDK
- Benefits: bypass CPU & kernel, zero-copy (between NIC, kernel and user-space)
- Limitations: RDMA NIC caches the RDMA connection informaion (e.g., rkey, address) and queues of each local-remote pair on the NIC DRAM which has a limited size. So, the RDMA performance may go down with an increasing number of RDMA connections. In this case, 2-sided RDMA is preferred as no connection information is needed.


# SmartNICs (46:00)
### Type1: SoC (System on Chips) SmartNICs
- NIC with general purpose processing cores on the chip
- we can write arbitrary programs and upload them to the NIC
- On top of that, the NIC has several accelerating functions (RNG, encryption/decryption, (de)compression, Regex matching) which can be called during the pkt processing in our arbitray programs
### Type2: FPGA-based SmartNICs
- NIC with a big FPGA chip (instead of general-purpose cores)
- Microsoft Catapult is one example 



# Operating Systems
- Unlike normal software, OS is event-driven
- That is, OS is executed only whenever an event occurs 
- If a running process (in user space) invokes an event, the OS is executed in kernel space with higher privilege level, 
and return back to the user space when its done  

# Categories of Events
### Hardware Interrupts
- Raised by external HW devices (e.g., NIC when it receives a pkt, or keyboard, mouse, USB etc.)
- These are asynchronous and may occur at any time
### Traps
- i.e., SW interrupts
- Raised by user program to invoke some OS functionalities (e.g., system call)
### Exceptions
- generated automatically by the processor itself as a result of an illegal instruction
- 1) Faults: recoverable errors (e.g., page fault)
- 2) Aborts: difficult to recover (e.g., divide by 0)

# How HW Interrupts are Handled?
### Single Device
- In general, today's processors have a dedicated interrupt pin (called INT)
- For example, a keyboard is connnected through INT, and when a particular key is pressed, an iterrupt is generated to the processor.
- Then, the processor is switched to 'Interrupt Handler Routine', and executes some instructions
- Once the handler is finished, the process is switched back to the original program
### Multiple Devices


##### Physical Layer: modules (implemented in a hardware called PHY-chip) enabling two nodes' communication by 1) Encoding (digital => analouge); 2) Upon receiving analogue, decoding (analouge => digital);

##### References
[1] https://www.youtube.com/watch?v=rnGVincwk30
