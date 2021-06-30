---
title: "C++ - Multi Threading"
date: 2021-06-25
categories:
  - C++
tags:
  - multi-threading
---

- th1.join(): current thread waits here for th1 to finish
- th1.detach(): th1 will be freed as its own -- become a deamon process
- If th1 is destroyed (e.g., th1 is created in main, and main ends), then the program will terminate th1
- cout << std::thread::hardware_concurrency() << endl : to see the number of threads can be running concurrently 
(i.e., # of logical cpu cores -- "$ sysctl -n hw.logicalcpu" on terminal)

# data race
- a situation that the outcome of program depends on the relative execution order of threads  
- typically bad for the program, so we should avoid it
- how to solve? mutex!

### mutex
- assume: std::mutex mu1;
- to synchronize the access of the common resource 
- As a simple way, we can use mu1.lock() and mu1.unlock() before and after the line accessing common resource or data (e.g., variables, cout, etc.)
- However, if some error happens in between, mu1 can be locked forever, which is bad
- So, we use std::lock_guard<std::mutex> guard(mu1);
- Then, whenever the guard goes out of scope, mu1 will always be unlocked

##### References
[1] https://www.youtube.com/watch?v=f2nMqNj7vxE&list=PL5jc9xFGsL8E12so1wlMS0r0hTQoJL74M&index=2
