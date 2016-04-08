---
layout: post
title: 'Cassandra ThreadPool statistics'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Cassandra uses a lot of thread pools under the covers. You can get information
about them with the following metrics:

```
o.a.c.m.ThreadPools.{request, internal, transport}.*
```

There are a lot of them and are grouped into three categories:

* request - used directly by your reads and writes
* transport - threads used to deal with connections over the native protocol (CQL)
* internal - background tasks like flushing memtables to disk and gossip

The first part of `nodetool tpstats` output:

```
Pool Name                    Active   Pending      Completed   Blocked  All time blocked                                                                                                                   [9/1984]
MutationStage                     0         0             50         0                 0
ReadStage                         0         0              0         0                 0
RequestResponseStage              0         0              0         0                 0
ReadRepairStage                   0         0              0         0                 0
CounterMutationStage              0         0              0         0                 0
MiscStage                         0         0              0         0                 0
HintedHandoff                     0         0              0         0                 0
GossipStage                       0         0              0         0                 0
CacheCleanupExecutor              0         0              0         0                 0
InternalResponseStage             0         0              0         0                 0
CommitLogArchiver                 0         0              0         0                 0
CompactionExecutor                0         0             23         0                 0
ValidationExecutor                0         0              0         0                 0
MigrationStage                    0         0              0         0                 0
AntiEntropyStage                  0         0              0         0                 0
PendingRangeCalculator            0         0              1         0                 0
Sampler                           0         0              0         0                 0
MemtableFlushWriter               0         0              7         0                 0
MemtablePostFlush                 0         0             23         0                 0
MemtableReclaimMemory             0         0              7         0                 0
Native-Transport-Requests         0         0              0         0                 0
```

If your not averse to reading
[Javadoc](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html) then understanding how the
`java.until.concurrent.ThreadPoolExecutor` (`TPE`) works will help you understand
`tpstats` as many of the columns map to concepts described in the high level
description of how the `TPE` works. Essentially a `TPE` is a pool
of threads and a queue for when all the threads are in use.

The columns:
 
 * **Active**: currently running task. This maps to a call to `getActiveCount()` on
   the underlying `java.util.concurrent.TreadPoolExecutor`. 
 * **Pending**: On the queue of the `TPE`
 * **Blocked**: Casandra configures the `TPE` to block if all threads are in use and
 the queue is full

Why does it block rather than shed load? Well that would block the work that
could succeed before a timeout as it is the newest. Casandra sheds load when a
task is attempted to be run if its creation time is older than the timeouts you
configure in `cassandra.yml`.

## ReadStage and MutationStage

It probably goes without saying that Pending is bad and blocked is worse. That
is not to say that they should always be 0. Take `ReadStage` and
`MutationState`. Ideally I want Pending to be 0 but if a spike of writes
or reads are sent to a Cassandra node of course it will get behind! But assuming
that your `read_request_timeout` and `write_request_timeout` is being met your
read/write will succeed. The issue if these figures are regularly high and you
start to miss your SLAs. This
means your Cassandra cluster is constantly getting behind and either some tuning
is required or you need to scale your cluster.

To size of the thread pools is set with the `concurrent_reads` and
`concurrent_writes` properties and at the time of writing  default to 32 if not
set.

## MemtableFlushWriter

This does what you would think. When a memtable is ready to be flushed to disk it
isn't done synchronously in a request it is done in the background by this
'TPE'. I find that any Blocked requests for this indicates that your disks can't
keep up with your write load. This will default to the number of disks you have
configured for Cassandra data directory. Which makes sense for spinning disks
but if you're using SSDs you can typically increase this by setting the
`memtable_flush_writers` property to allow concurrent flushes to disk.

## Native-Transport-Requests

TODO

## The rest

If there are any other thread pools you'd like me to discuss then leave a
comment. The above are the ones I watch the most. 

If you want to look into a different thread pool I tend to start at
`o.a.c.c.StageManager` where all the thread pools are created. 

```
stages.put(Stage.MUTATION, multiThreadedLowSignalStage(Stage.MUTATION, getConcurrentWriters()));
stages.put(Stage.COUNTER_MUTATION, multiThreadedLowSignalStage(Stage.COUNTER_MUTATION, getConcurrentCounterWriters()));
stages.put(Stage.VIEW_MUTATION, multiThreadedLowSignalStage(Stage.VIEW_MUTATION, getConcurrentViewWriters()));
stages.put(Stage.READ, multiThreadedLowSignalStage(Stage.READ, getConcurrentReaders()));
        
```

The construction of each of the stages pass on the max size thread pool size e.g.
`getConcurrentWriters()`. It typically results to a call to `o.a.c.c.Config`
which is a java class that matchs the structure of `cassandra.yml`.
