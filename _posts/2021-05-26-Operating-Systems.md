---
title: "Operating Systmes"
date: 2021-05-13
categories:
  - OS
tags:
  - OS
---



# How process (or program) works?
##### Life-cycle of a program
- I write a program *hello.c* (high-level)
- *hello.c* is given to the compiler
- Compliler convert the high-level language to a low-level language, *a.out* (executable on the machine)
- Operating system does not take high-level language, but it can understand low-level language to run on HW
- Here, *hello.c* and *a.out* are **programs** residing in the hard-disk memory i.e., secondary memory.
- Now, OS takes a program and put it in the RAM (main memory) to execute
- When a program is put on the main memory, it will create a data structure ==> This is the **process**
- **Process**: something created by the OS in order to execute a **program**.

##### Process in the main memory (the date strucutre)
- Stack: for function call (grow downwards)
- Dynamic allocation (i.e., heap): to be used by the process at some point when needed (grow upwards)
- Static variables (variables remained forever thoughout the life-time of process)
- Executable code (*a.out*)
------------
- On this structure, CPU takse the lines in *a.out* one by one to execute, while referring some points in the above memory on demand
- OS make sure to never access beyond the boundary of the process in the memory (otherwise, segmentation fault) 

##### Attributes of a process (maintained in PCB of the process)
- Process ID
- Program Counter: what is the next instruction to be executed?
- Process State: ready, running, blocked, etc.
- Priority
- General Purpose Registers: registers are multiplexed by multiple processes, so a preempted process need to store the state of them so that it can restore them again when it preempts again
- List of Open Files: to keep it across preemptions
- List of Open Devices: to keep it across preemptions
- Protection: prevent a process from accessing other processes' data
* OS maintains the linked-list of PBCs


##### Two types of time of a process
- CPU time
- I/O time

# Introduction
### What is OS?
- Interface between user and HW (help to generate system calls as user wants)
- Resource allocator (resource should be shared by many users or programs, or should be scheduled if not shareable)
- Manager for memory, process, files, security (e.g., resource isolation), etc.

### Goal of OS
- Convenience (e.g., for personal computer)
- Efficiency (e.g., for servers)

### Type of OS
##### Batch OS: 
- Single computer, queue of jobs (FIFO),
- Can move on to next job iff the previous job is finished (starvation: waste of idle resource e.g., CPU or I/O), low efficiency
- Not interactive since an interactive process has to wait until all previous jobs are finished 
##### multi-programming
- When a job is taking I/O resource but not CPU, the other job can take CPU (i.e., no CPU idle time)
##### multi-programming + preemption: multi-tasking
- CPU can be multiplexing multiple jobs even before completing them (i.e., preemptive) 
- Interactive
##### multi-processing
- Have multiple CPUs
- Many processes can be supported simultaneously
- Differences from multiple computers: other resources (memory, etc.) can be shared, so less costly and more efficient
- Throughput improved
- Higher reliability (tolerant to some CPU failures)
- Each CPU can be operated as we want (batch? multi-tasking?)
##### Real-time
- Requested jobs will be having some deadlines
- If fails to meet the deadline, results can be discarded



##### References
[1] https://www.youtube.com/watch?v=2i2N_Qo_FyM
