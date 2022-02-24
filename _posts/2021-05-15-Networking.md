---
title: "IRQ"
date: 2021-05-12
categories:
  - OS
tags:
  - IRQ
---


# NIC
- a device (a piece of peripherals) to enable a computer to communicate with other devices in the network
- connects to a system bus (mostly attched to a PCIe bus)
- multiple types of NICs: Ethernet, Wireless, InfiniBand (similar to ethernet but used in HPC)
- getting faster (100 GbE alreday) and more functional
- ASIC

### NIC Driver
- computerized instructions and information requird by NIC to operate
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
