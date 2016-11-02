---
layout: post
title: "Using perf to spy on a docker container's system calls"
author: Christopher Batey
comments: true
tags:
- docker
- perf
---

I'm a developer and now accidental platform person who now manages a lot of servers 
all running Kubernetes/Docker and sometimes I wonder what on earth all the containers are up to.

Well perf can tell us which system calls are being made at a cgroup level. 

Added [here](https://lwn.net/Articles/421574/) back in 2011 by Google.

This will work for rkt and any thing that is managing multi tenancy with cgroups.

How does it work?

Docker puts the processes in a container in their own cgroup. The Docker id you see
from docker ps is the prefix of the group name. E.g.

```
➜  ~ docker ps                                                                                                                                                                     
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
50993dd8689e        a48ba3695664        "/sbin/my_init -- /st"   12 minutes ago      Up 12 minutes       0.0.0.0:8080->8080/tcp, 8081/tcp   loving_morse
```

You can see that with `systemd-cgls` as well:

```
-.slice
├─docker
│ └─50993dd8689e561467739465a5491f0cf42a671783d4ad24870278118ee88149
│   ├─2399 /usr/bin/python3 -u /sbin/my_init -- /start_java.sh -c -jar /data/app.jar server /data/app-config.yml
│   ├─2426 /usr/bin/runsvdir -P /etc/service
│   ├─2431 /bin/bash /start_java.sh -c -jar /data/app.jar server /data/app-config.yml
│   └─2443 java -XX:NativeMemoryTracking=summary ...
```

Or from the cgroup file system:

```
➜  ~ ll /sys/fs/cgroup/cpu,cpuacct/docker/                                                                                                                                         
50993dd8689e561467739465a5491f0cf42a671783d4ad24870278118ee88149/  cpu.cfs_period_us                                                
cgroup.clone_children                                              cpu.cfs_quota_us                                                 
cgroup.procs                                                       cpu.shares                                                       
cpuacct.stat                                                       cpu.stat                                                         
cpuacct.usage                                                      notify_on_release                                                
cpuacct.usage_percpu                                               tasks    

```

Okay so the cgroup for this container is `docker/50993dd8689e561467739465a5491f0cf42a671783d4ad24870278118ee88149/`

To see what it is up to we can run perf on the host with that cgroup and the events we want to
capture. Perf stat can record system calls and here is how we do it:

```
git:(master) ✗ sudo perf stat -e "syscalls:*" -G docker/50993dd8689e561467739465a5491f0cf42a671783d4ad24870278118ee88149 -a

               257      syscalls:sys_enter_sendto                                   
               257      syscalls:sys_exit_sendto                                    
                98      syscalls:sys_enter_recvfrom                                   
                98      syscalls:sys_exit_recvfrom                                   
                 4      syscalls:sys_enter_setsockopt                                   
                 4      syscalls:sys_exit_setsockopt                                   
                82      syscalls:sys_enter_sendmsg                                   
                82      syscalls:sys_exit_sendmsg                                   
             5,720      syscalls:sys_enter_recvmsg                                   
             5,732      syscalls:sys_exit_recvmsg                  
... // lots more output
```

Happy spying.

