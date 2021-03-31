---
title: "Linux - Network Namespace"
date: 2021-03-28
categories:
  - Linux
tags:
  - network
---

## Network Interface
- Definition: Interface (SW or HW) between a device and a network it is connected to 
- Each interface has its unique address (usually IP address)
- loopback (127.0.0.1) is a special interface of the network consist of a single device
- 0.0.0.0 is another special interface representing EVERY interfaces of the device (bad security) 

##### What does it actually mean?
- Since an interface is a connecting point of the device and a specific network, other devices outside the network cannot connect to the interface. 
- For example, if an app is listening to the loopback interface, only other apps running on the same device can access to it. 
- Also, requests have to be sent to the correct interface the app is listening to

## Network Namespace
- To allow multiple isolated network environments (namespace) running on a **single** physical host (or VM)
- Each namespace has its own interfaces, routing & forwarding tables and network security isolation
- Processes can be dedicated to an individual namespace (to separate them from other namespaces)
- Used in Mininet, Docket, ...

## Basic Commands
- Check network interfaces
```
$ ip link
or
$ ifconfig
```
- Check address
```
$ ip a 
```
- Check all connections established to the device
```
$ netstat
Ex. to check Tcp connections (i.e., sockets) that the device is Listening in Numbers
$ netstat -tln
```
- Check routing table
```
$ ip route
```

## Namespace Handling
##### Add namespace
$ ip netns add red
##### Root Namespace



##### Reference
https://www.youtube.com/watch?v=PYTG7bvpvRI
https://www.youtube.com/watch?v=_WgUwUf1d34


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
