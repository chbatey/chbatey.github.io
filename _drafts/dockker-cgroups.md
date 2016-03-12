---
layout: post
title: 'Docker vs the JVM: CGroups'
author: Christopher Batey
comments: true
tags:
- docker
---


Docker offers great flexibility when it comes to sharing out the resources of a
single host. You can:

* Memory: Absolute value for the memory. The container will be killed if it
tries to use more. 
* CPU Shares: 
* CPU set: Pin the container to a set of CPUs

 [1] https://docs.docker.com/engine/reference/run/
 [2] https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/
