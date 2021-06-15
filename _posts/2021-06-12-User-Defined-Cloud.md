---
title: "User-Defined Cloud"
date: 2021-05-13
categories:
  - Paper Review
tags:
  - UDC
---

# Talk Summary
- Cloud Computing (since 2006 of AWS release) has evolved from a simple service (running computers) to a sea of services
- Today, AWS has 175 services, and EC2 alone has 331 instance types

### Common pattern of cloud compuing evolution
- A provider identifies a new type of popular application or HW
- He develops SW or HW infrastructure for these trends
- He launches a new service or extends an existing one 

### Use-case
- Take an example where a hospital is trying to move its digital data and computation to the cloud
- Hospital first wants to migrate its patient medical records to the cloud *securely and with privacy*
- Then, it performs some data analysis (*either on-demand or real-time*) to get certain insights 

### Problems in the today's cloud
- No on-demand (serverless) accelerator: needs to maintain VMs with accelerators instead, which causes extra cost when there is no demand
- No secure accelerator: needs to use secure CPU, which causes performance degradation
- No find-grained turning of system features: 
the user may need to use different consistency levels and replication factors for different parts of the application  

'''
Today's cloud doesn't have right services for niche applications, so users have to build a local cluster or use a third-party service
'''

### Why it happens?
- Disconnection between cloud providers (who design and define the cloud) and the users (who know what is needed)

### User-Defined Cloud
- to give conrols to users to define the computing resources and their features for their own workloads 
- but still cloud providers supply SW and HW infrastructures under the hood (to avoid IT burden to the users)

### How it works?
[User Side]
- Application developer team of the user side decide the application semantics
- IT team look at the application and decide a set of specifications (HW & SW, execution environment, security specification, distributed semantics)
[Provider Side]
- manage and provide SW and HW infrastructures

### How can the provider achieve user's specification?
- Idea: to develop HW and SW layers in a fine-grained way and put them together on demand from users

##### References
[1] https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s02-zhang.pdf
[2] https://www.youtube.com/watch?v=bd-06tw769o
