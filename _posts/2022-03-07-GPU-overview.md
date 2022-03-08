---
title: "GPU overview"
date: 2022-03-07
categories:
  - System
tags:
  - GPU
---

NUMA is a system that is used in computers with multiple physical CPUs.

# GPU
- originally designed for 3D games (to support graphics consist of more polygons or pixels)
- but recently basically doing everything (supporting intensive computation power)

### Early Architecture (in 2010s)
- Handle multiple tasks: ex. vertex processing, rasterization, pixel processing, etc.
- Task Parallelism: independent tasks in parallel using pipeline
- Data Parallelism: each task in parallel with independent data 
- Configurable but not very programmable

### More programmability
- Gradually more demand on (user-specified) customized processing
- still have issue of **load-balancing**:
  * pipeline design => overall performance is bottlenecked by the slowest stage
  * if we use a complex processing in a stage, overall performance is restricted by that
- Shader Model 4.0 (to deal with load-balancing issue): unified instruction set for all stages (modern GPU architecture)
  
  
### CUDA: Compute Unified Device Architecture
- Interface between general programs and GPU
- Nvidia's SW framework for GPGPU (including libraries, compiler, API)
- Supports many languages (C, C++, Fortran, OpenCL, ...)
- To express parallel general-purpose computations
- Two components:
  * Host program: sequential threads on CPU
  * Kernels (not the OS kernels): lightweight parallel threads on GPU

##### CUDA Kernel Thread
- is a single thread running sequentially (conceptually same as the x86 CPU thread, but weaker)
- each thread has a thread-local memory (not accessible by other threads)
- Warp: a group of 32 threads with the same instructions

##### Hierarchical Thread model
- A thread (or warp)
- Thread block (consist of 512 or 1024 threads): synchronize via barriers, communicate via per-block shared memory
- Grid: consist of multiple thread blocks, communicate via per-application global memory, different kernel grids synchronize using global barriers
* what is the 'barrier'? It's a synchronization point where all threads are temporarily blocked until all of them are synchronized



### Modern GPU Architecture
- Massive data parallel architecture with unified intructions (not task-parallel pipeline anymore)
- start of GPGPU era, each of many cores (acts like a weak CPU core) can perform different instrucutions on different data (flexibility)
- has its own on-die DRAM with **high memory bandwidth**
Components:
  - DRAM interface: GPU cores access the on-die DRAM through this
  - Host interface: PCIe interface to the host CPU or other PCIe devices  
  - 16 streaming multiprocessors (SMs) run thread blocks
  - Each SM has 32 CUDA cores
  - Global L2 Cache (Shared by all SMs)
  - GigaThread block: schedule threads blocks to SMs

##### Streaming Processr
- 32 CUDA cores
- 16 load/store units
- 4 special function units
- 64KB shard memory and L1 cache
- Can run up to 1536 concurrent threads (since the memory BW is high, good to run many concurrent threads to hide memory access latency)
- Schedule in the warp granularity 

###### CUDA core and SFU
- 1 CUDA core runs 1 thread at a particular time
- CUDA core has floating point unit and interget unit
- SFU (special function unit) proivdes fast approximation for 1/x, square root, sin, cos, exp, logarithm, etc.
###### Load/Store Unit
- perform load, store and atomic memory operations
- it can access three different memory regions
  * per-thread local
  * per-block shared
  * global memory

### Lifetime of a GPU program
- transfer data from host memory to GPU memory (via host interface i.e., PCIe)
- host program instructs GPU to launch kernel 
- parallel execution on GPU
- transfer result from GPU memory to host memory (via host interface again)
==> overall, long latency process

*Summary: GPU is highly optimized for throughput, but not latency*

# GPU Programming
- API: from configurable (fixed) graphics processing to general purpose
- OpenGL: programmable shader, but still a graphichs API
- Now, GPGPU (CUDA, OpenCL)
  * Fully programmable
  * Dynamic control flow
  * read/write to local and global memory


### Good performance requires good understanding of the GPU HW
- Branches are not free
- minimize communication and synchronization between threads
- GPU memory hierarchy
- CPU-GPU co-processing (how to divide tasks, when to perform each task




##### References
