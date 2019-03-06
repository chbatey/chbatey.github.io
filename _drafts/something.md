---
layout: post
title: 'Making the most of a JVM inside a Docker/Kubernetes'
author: Christopher Batey
comments: true
tags:
 - kubernetes
 - efficiency
 - jvm
---

A less cited benefit of Docker/Kubernetes is it can encourage software efficiency.
Previously unless there has been a business need for effciency it has been overlooked
as the resources the physical machines or VMs our JVMs have been running on are
either fixed or it is slow to change. Meaning that any increase in effeciently won't
materialize into cost savings until hardware is retired or VMs can be re-provisioned.

Only when a service needs to be horizontally or vertically
scaled does the cost of inefficiency become apparent by which time the
architecture of the application may be impossible to change.

Docker/Kubernetes enable multi tenancy and seemless moving of resources along with instant
financial gains from increasing the effeciency of the software running on it. 

How do we make our JVM based applications effecient when we're used to running JVMs with GBs of
heap and hundreads, possibly thousands, of threads?

The answer depends on whether your application is CPU bound, IO bound, memory bound which often manifests its
self as "Concurrent request bound". This article focuses on the latter
but touches upon CPU bound.

If your application is CPU bound:

*  CPU limits vs CPU request

See profiling. 

If your application is memory bound or uses an unbounded amount of memory depending on
the number of concurrent requests then see using flow control for a constant memory footpring.


## Reduce number of OS threads with an Async programming model

## Constant memory footprint with application level flow control

## Profile to increase effecieincy effecient



