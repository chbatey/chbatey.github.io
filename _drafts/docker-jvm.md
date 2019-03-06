--
layout: post
title: 'JVM with Docker and Kubernetes in production'
author: Christopher Batey
comments: true
tags:
- docker
---

> Running something inside Docker is easy

Said no one that has tried running a JVM inside Docker
in a multi tenant production environment. 

Having spent years running JVMs inside Docker via Kubernetes
I can tell you that is isn't straightforward. Developers
need a whole host of additional knowledge. It isn't easy
to know upfront what that knowledge is or that any is required
as Docker has received a mass of developer marketing saying
it is easy as running a few commands.

I now build JVM libraries for a living with the aim is to make
them easy to run inside a container and that also isn't 
plain sailing.

In this article I'll discuss:
* The history of JVMs inside containers because we're not all on JDK 11 are we?
* What developers need to know to be successful
* Memory issues 
* CPU issues
* Base images

## History

## Need to know

### CPU allocations in containers

* Specifically in Kubernetes

### Memory limits

### Disk access

Default file systems?

## Memory related issues

### -XX:+UseCGroupMemoryLimitForHeap

### -XX:+UseContainerSupport

### -XX:+PreferContainerQuotaForCPUCount

## CPU related issues

## Shared memory, RAM disks, etc

[] https://www.youtube.com/watch?v=6ePUiQuaUos
