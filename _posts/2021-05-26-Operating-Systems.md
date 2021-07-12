---
title: "Operating Systmes"
date: 2021-05-13
categories:
  - OS
tags:
  - OS
---

### Terminologies
- Preemption: forcefully pause a running process (with PCB stored for resume later) and take the next ready process to the CPU 
- Context Switching: Performed by dispatcher. Changing the process context (PCB) and state of two processes (1 Ready and 1 Run) for the preemption.
- (CPU) Time Quantum: a time slot (duration) assigned to a process to use CPU (once expired, preemption happens)
- Degree of Multi-programming: the max # of processes in ready that RAM can hold at a time

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
- I/O time (I/O can be perforemd w/o RAM, so the process can be in disk during this time)

##### States of a process
- New (when the process is being created by OS, => program is being taken from disk)
- Ready (once the creation of process is completed, # of ready processes RAM can hold at a give time is limited => process is on the RAM)
- Run (the ready process has been chosen out of other ready processes according to the strategy, and is running => process is on the RAM)
- Block or Wait (when the process is in I/O time or waiting for other resources currently unavailable, so doesn't need for CPUs. Once finishes I/O or using other resources, go back to Ready state. => process is on RAM)
- Termination or Completion (once the process is done, the process will be killed and it's all traces like PCB context are deleted => process is deleted from RAM)
- Suspend Ready (the process was in Ready, then a new important process is created making RAM reaches degree of multi-programming, and this process is the least important according to the scheduling algorithm, so it's kicked out to the disk and is waiting to be restored => process on disk)
- Suspend Wait or Suspend Block: (similar as Suspend Ready, but kick out Block state process instead of Ready, which is more reasonable => process on disk)
------------
State transitions happen by OS or intrinsic logic of the process

##### Operations on process
- Creation: the process is placed on RAM and becomes ready
- Schedule (dispatch): the process is chosen and assigned to use CPU
- Execute: the process is executed by CPU
- Killing/Delete: the process is over

##### Time parameters of a process
- Arrival Time: time at which the process enters into the ready state (ready queue)
- Burst Time: the amount of CPU time required by the process to be completed (it's practically difficult to estimate before execution)
- Completion Time: time at which the process finishes
- Turn Around Time: Completion Time - Arrival Time
- Waiting Time: time during which the process is waiting (to use CPU) in the ready queue or wait state 
- Response Time: the first time at which the process is scheudled (dispatched) to CPU == duration of the first Waiting Time
-----
Turn Around Time = Completion Time - Arrival Time = Total Burst Time + Total Waiting Time


# Schedulers
### 3 types of schedulers in state transitions 
All of them affects performance
##### Long-term: 
- At *New* state. 
- How many processes should be created (this decision lasts for the entire process).
- It decides the degree of multi-programming. 
- Affects system performance significantly (should balance CPU-bound and I/O-bound processes for better performance). 
##### Short-term: 
- At *Ready* state
- which ready process to schedule (this decision lasts only during the *Run* state). 
- Then, dispatcher handles the context switching.
- Good short-term scheduler make the decision such that minimizes the context switching time (which depends on the size of context to be changed)
##### Medium-term: 
- At *Ready* or *Block* state, 
- Whether to suspend a process or not
- Suspension of a process cause *swapping* -- moving a process from RAM to disk, and getting it back
- Good midium-term scheduler minimizes the swapping overheads

### CPU scheduling
- What? to pick a ready process and allocate it to CPU
- Who? short-term scheduler
- Where? from ready to running state
- When? a process moves..
  - run -> terminate (always)
  - run -> wait (always)
  - run -> ready (due to quantum expiration or a new process with higher priority) (always)
  - new -> ready (not always)
  - wait -> ready (not always)

### Scheduling Algorithms
##### FCFS
- Criteria: arrival time
- Mode: non-preemptive




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
##### multi-programming w/o preemption
- It has multiple processes in 'Ready' state
- When a job is taking I/O resource but not CPU, the other job can take CPU (i.e., no CPU idle time)
##### multi-tasking: multi-programming with preemption
- Also called Time Sharing
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
