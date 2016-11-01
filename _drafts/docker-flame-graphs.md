---
layout: post
title: 'Flame graphs and perf-top for JVMs inside Docker'
author: Christopher Batey
comments: true
tags:
- docker
- perf
- flame-graphs
---

Perf top and flame graphs have become part of my daily tool chain.

Most of my "real" JVMs, or at least the ones I get paid to tinker with, run inside Docker containers running inside Kubernetes
which makes using perf more challenging.

If you haven't used flame graphs and perf with Java see Brendon Gregg's
[post](http://www.brendangregg.com/blog/2014-06-12/java-flame-graphs.html)
and a talk Nitsan Wakart's talk at [Devoxx UK](https://www.youtube.com/watch?v=7PkkxDaFDj8) on the topic and come back armed with
knowledge of perf-map-agent and perf.

Right ready for some beautiful flame graphs?

![Example flame graph]({{ site.url }}/assets/flame_graphs/flamegraph-4225.svg)

So perf-map-agent generates flame graphs and allows perf top for Java programs
by:

* attaching to the JVM and getting a symbol map
* running perf top of perf record with the map file

When our JVM is inside a Docker container it's inside its own PID namespace
typically running as a user that only exists inside the container.

This means that when we look at the JVM from
the host it will have a different PID than the same process inside the container.

To attach to a JVM and get the symbol map the jvmstat file (files in
`/tmp/hsperfdata_<user>/<PID>`) is used, that will
be in the container tmp directory named after the container PID. Typically only
the user that owns the JVM can attach to the JVM.

When we run perf from the host it'll look (by default) for a mapping file in the
host tmp directory (`/tmp/perf-<PID>.map`).  We don't run anything as root in the container so we
certainly can't run perf in there without creating a gaping security hole.

Perf map agent without containers puts the map file in the expected location.

So to make this work:

* have the perf map agent mounted inside each of our containers (I have it on
  every server and mount the volume read only)
* get the symbol map from inside the container using the container PID and copy it to the host tmp
  directory. Renaming it from the container PID to the host PID.
* run perf on the host as root to get top or a flame graph

Here's it in action:

Finding the process on the host:

```
➜  docker-jvm-perf ps -ef | grep dropwizard-app | grep " java "                                                                                                                                                   
root     20727 20722 11 20:10 ?        00:00:05 
java -XX:NativeMemoryTracking=summary -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysPreTouch -XX:+UseG1GC -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGC -jar 
/data/dropwizard-app-all.jar server /data/config.yml
```

You can also use tools like `systemd-cgls` to see which processes are in each
docker containr. The group name will be the same as your container id.
Or `docker inspect` if the JVM is the PID 1 inside the container (hint, it
shouldn't be). xxx include link to PID 1 problem

The host PID here is `20727`.

Running ps inside the container (so in the PID namespace of the container):

```
docker-jvm-perf docker exec ca94c4ffdb3f ps -ef | grep java                                                                                                                                                    
root         1     0  0 20:10 ?        00:00:00 /usr/bin/python3 -u /sbin/my_init -- /start_java.sh -c -jar /data/dropwizard-app-all.jar server /data/config.yml
root        10     1  0 20:10 ?        00:00:00 /bin/bash -x /start_java.sh -c -jar /data/dropwizard-app-all.jar server /data/config.yml
root        15    10  2 20:10 ?        00:00:05 java -XX:NativeMemoryTracking=summary -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysPreTouch -XX:+UseG1GC -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGC -jar /data/dropwizard-app-all.jar server /data/config.yml
```

The PID inside the container is `15`. Don't be fooled, this is the process we
are looking for, it is just the linux PID namespace in action.

We can see the jvmstat files inside the container:

```
➜  docker-jvm-perf docker exec -it ca94c4ffdb3f bash                                                                                                                                                              
root@ca94c4ffdb3f:/# ll /tmp
total 0
drwxrwxrwt.  3 root root  29 Oct 31 20:10 ./
drwxr-xr-x. 22 root root 287 Oct 31 20:10 ../
drwxr-xr-x.  2 root root  16 Oct 31 20:10 hsperfdata_root/
root@ca94c4ffdb3f:/# ll /tmp/hsperfdata_root/15 
-rw-------. 1 root root 32768 Oct 31 20:15 /tmp/hsperfdata_root/15
```

I have a very noddy gradle project that will build a docker image with 
a JVM app: xxx

It is a fantastic web application that has an endpoint that calculates all
the primes up to a given number (ignoring all efficient ways of doing it).

So given a container ID and a host process ID we can run:

```
➜  bin git:(docker-support) ✗                                                                           
➜  bin git:(docker-support) ✗ ./docker-perf-top 6ee59c68e1f9 24554     
```

You can see C1 and C2 kick in if you run this a few times (each time generates a
new symbol map which will be updatd based on what the JIT has been up to). 

Quickly the isPrime method gets inlined away (see Nitsan's talk on how to see
inlined methods in perf if that is what you desire).


And then for a flame graph:

```
➜  bin git:(docker-support) ✗ ./docker-perf-java-flames 6ee59c68e1f9 24554   
```

Unsurprisingly we spend all our time in callout as it is busy calculating our
primes.

xxx missing symbols if jvm, glibc and pthread is not on the host.


Here's my fork of perf-map-agent with the docker perf map scripts:

It assumes:

* perf-map-agent is mounted inside the container at: xxx
* you have root access on the container host
* Jps is on the path inside the container
* you have a single JVM inside the container

  Happy profiling!


