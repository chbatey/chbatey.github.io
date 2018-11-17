---
layout: post
title: 'The JVM in Docker 2018'
author: Christopher Batey
comments: true
tags:
- docker 
- jvm 
- kubernetes
---

Threeish years ago I worked with JVMs inside Docker running in Kubernetes. I discussed some of the pain points 
in a conference talk you can watch [here](https://www.youtube.com/watch?v=w1rZOY5gbvk), but you shouldn't as it is
out of date.

Since then the JVM has moved on. We now have:

* `-XX:+UseCGroupMemoryLimitForHeap`
  * Added in the later JDK8 patch releases where the `cgroup.limit_in_bytes` is used for JVM ergonomics 
  * Removed in JDK10 with the behaviour becoming the default and can be turned off with `UseContainerSupport`.

* `-XX:+UseContainerSupport`
  * Added in JDK10. Enables the memory support as well as the CPU support discussed below

* `-XX:+PreferContainerQuotaForCPUCount`
  * Added in JDK11. Support for using the `cpu_quota` instead of `cpu_shares` for picking the number of cores the JVM uses
    to makes decisions such as how many compiler theads, GC threads and sizing of the fork join pool. 


## CPU for JVMs inside containers

The JVM  makes decisions based on the number of cores, such as:

* Compile threads 
* GC threads
* Fork join pool size

In addition libraries use  `Runtime.availableProcessors` to size thread pools.
Historically when running in a container all of these would need to be overridden. 

### Can the JVM just 'use" the docker CPU allocation instead?

The short answer is `it depends`. There is no right answer, it depends on your infrastructure and application.

There are three ways a Linux container can have its CPU limited:

* `cpu_shares`

A relative figure that gives the container a share of the CPU. Default value is 1024. For example
if one container has 512 and another has 1024 then the second container will get double the CPU time
as the first. However if there is no contention e.g. container one doesn't use any CPU time then the
second container can use all available CPU cycles.

`cpu_shares` allow the true benefits of increased utilisation containers can offer by being fair when
all the containers need CPU time but still allow a host to be fully utilised even if some of the containers
don't need any CPU time.

* `cpu_quota`

A hard CPU time that can be used per `cpu_period`. The default period is 100 microsecond so setting this to
50 microseconds is `kind of like` giving a container half a core and setting it to 200 microseconds is `kind of like`
giving a container 2 cores. However if the host has more cores than this and your application is multi threaded then
the threads can run on many cores and use up the quota in a fraction of the time. E.g. a host with 64 cores and an 
application with 20 threads that have work to do can use up a quota of 200 micoseconds in 10 microseconds meaning
all 20 threads will throttled for 90 microseconds. You can track this with the `throttled_time`
in `cpu.stat`.

I don't think you should ever use `cpu_quota` for a JVM application.


* `cpu_sets`

Run the container on specific CPUs. Not discussed as not used in the mainstream container orchestration market.


### What does the JVM with `ContainerSupport`

JDK10 divides `cpu_shares` by 1024 to pick a number of cores. JDK11 first looks at `cpu_quota`, if this is set then it is
divided by `cpu_period` to get an effective number of cores. If not set then the `cpu_shares` are used like with JDK10.

Is this a good idea? I think the qutoa support is the right thing to do. If an application bases its thread pools 
on `availableProcessors` then the quota won't be used up by additional cores being available. Quotas are a hard limit
so there's no point having more GC threads or processing threads (unless you block them on IO) than the effective core 
count based on your quota.

However I'd never use `cpu_quota` for a JVM application because they nearly always have far more threads than
cores be them effective or real.

The `cpu_share` support is debatable as to whether it is a good idea. You need to evaluate your
application's threading model and decide if it will hinder utilisation.

Imagine a typical deployment of many
containers to Kubernetes. There will be many containers with JVMs running on each host. Previously they would
each have gotten to burst and use all the cores for say a big parallel GC and if all of them happen to do it
at the same time then they'd get a fair share. Now the GC threads will based on shares, which are typically set
quite low in these environments with the expectation they can burst. The same will happen for the ForkJoin pool
which is used for things like parallel streams. Before this support you'd instead end up with too many threads,
especially if the underlying host has many cores.

I'd expect to see utilisation in container clusters to drop with the defaults. 

### Running in Kubernetes?

Kubernetes sets `cpu_shares` based on `cpu.request` and `cpu_quotra` based on `cpu.limit`. For any version of Java
I'd avoid `cpu.limit` and for any JDK with the `cpu_share` support I'd set `cpu.request` to a much larger value that previously
otherwise you'll end up with too few GC threads and a ForkJoin pool that won't be able to fully utilise a host
when other containers are idle.









