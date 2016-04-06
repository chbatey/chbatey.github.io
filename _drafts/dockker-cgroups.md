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

My assumption is that you are running a JVM inside a Docker container so that
you can abstract away the physical machine with the goal of multi tenancy and
thus higher resource utilisation. If you fancy some light bed time reading I
recommend the Google Borg paper [4].

This post is about what you need to know to effectively set memory limits for
a JVM when running inside a container.

Rule 1: Do not accept the default HEAP settings

The JVM is clever, it doesn't have a random or hard coded value for the size of
your heap, it bases it off the amount of RAM on the machine. This and a host of
other auto configuration are something the JVM calls ergonomics and are
described in full in [3]. For a server class machine in Java 8 the initial heap
size is set to 1/64 of the total RAM and the maximum heap size is set to 1/4 of
the total RAM. This is only sensible if the JVM is expected to be the primary
user of the machine. For my current project we have machines with 64 GB of RAM
and have as many as 128 Docker containers running on a single machine. If they
had the default heap settings the JVMs could try and use 2048 GB of RAM. Not
good.

Rule 2: Test and verify to find the correct amount of memory for your container 

To handle multi tenancy we rely on Docker, or really CGroups under the covers,
to limit each container to a small portion of RAM. If the container uses more
than that amount of RAM it will be OOM killed. Coming up with the correct size
is tough as it is not just your heap. You also need to consider:

* Native memory
* Metaspace (PermGen replacement)
* Thread stacks
* Code cache
* Memory used for GC
* Internal JVM memory

For a good overview I suggest reading [5]. We don't have a set formula for this.
Some of our applications use libraries and frameworks that use a lot of native
memory via direct byte buffers and Unsafe so we've had to experiment with
realistic workloads.

Rule 3: Enable Native Memory Tracking (-XX:NativeMemoryTracking=summary)

This is to help with Rule 2. This allows you to use `jcmd` to track all the
memory the JVM is using which is invaluable for setting limits and working out
why your container was killed for using too much memory.

I workflow I often follow:

* Spin up my application in a container
* 
 

 [1] https://docs.docker.com/engine/reference/run/
 [2] https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/
 [3] https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html
 [4] http://research.google.com/pubs/pub43438.html
 [5] https://plumbr.eu/blog/memory-leaks/why-does-my-java-process-consume-more-memory-than-xmx
 [6] https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html
