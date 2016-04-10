---
layout: post
title: 'Docker vs the JVM: CGroups, controlling your CPU and memory'
author: Christopher Batey
comments: true
tags:
- docker
---


Docker offers great flexibility when it comes to sharing out the resources of a
single host. Lets start with memory:

* Memory: Absolute value for the memory. The container will be killed if it
tries to use more. 
* OOM Killing: By default your container will be killed if it reached its limit.
this can be disabled.

Docker delegates the enforcing of these values to the linux kernal's CGroup
feature. 


Next CPU:

* CPU Shares: 
* CPU set: Pin the container to a set of CPUs

[] https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/ch01.html
[] http://fabiokung.com/2014/03/13/memory-inside-linux-containers/
[] https://plumbr.eu/blog/memory-leaks/why-does-my-java-process-consume-more-memory-than-xmx
[] https://docs.docker.com/engine/reference/run/
[] https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/
