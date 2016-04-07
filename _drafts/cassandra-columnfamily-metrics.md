---
layout: post
title: 'Cassandra ColumnFamily metrics'
author: Christopher Batey
comments: true
tags:
- cassandra
---
Last time we looked at Cassandra's ClientRequest metrics. However they only take you so far as they are for all keyspaces and tables. Once you have identified a problem 
it is them time to zero in on a particular table. Assuming of course your issue is with Cassandra and not say the network connectivity between Cassandra nodes.

For this the metrics we want to look are are:

```
o.a.c.m.ColumnFamily.[keyspace name].[table name].{WriteLatency, ReadLatency, SSTablesPerReadHistogram, EstimatedPartitionSizeHistogram}
```

Remember the simplified three layers of Cassandra:

* Client APIs (Thrift, Native protocol for CQL)
* Dynamo 
* Storage

The ColumnFamily metrics are in the Storage layer in the `o.a.c.db` pacakge. They do not include any network time associated with meeting your consistency level.

The output from node tool looks similar to `proxyhistograms` but also includes
`SSTables`, `Partition Size`  and `Cell Count`

```
Percentile  SSTables     Write Latency      Read Latency    Partition Size        Cell Count
                              (micros)          (micros)           (bytes)                  
50%             1.00             11.86             61.21               258                 5
75%             1.00             14.24             73.46               258                 5
95%             2.00             17.08            126.93               258                 5
98%             2.00             20.50            263.21               258                 5
99%             2.00             24.60           1131.75               258                 5
Min             0.00              2.30             14.24               180                 5
Max             3.00         464228.84          74975.55               258                 5
```

# Read and Write latency


The `Write Latency` and `Read Latency` are what you'd expect. On this node 99% of
writes took less than 24.60 microseconds and 99% of reads took less than
1131.75 microseconds. These are very good figures and are from a high end SSD
running `cassandra-stress`. Remember the entire operations take much longer
than the above figures. 

This is just the storage layer of Cassandra and many of
these reads won't even have required disk access as it is all in the OS's page
cache. These are updated in the `o.a.c.db.ReadCommand` for reads and
`o.a.c.db.ColumnFamilyStore` for writes.


Spending time looking at the output from `nodetool cfhistograms` I started to
wonder why writes were always quicker than reads on Cassandra nodes that had
enough RAM for all SSTables to be in page cache. Well digging into the code here
is the snipper for the write latency point:

```java
/**
 * Insert/Update the column family for this key.
 * Caller is responsible for acquiring Keyspace.switchLock
 * param @ lock - lock that needs to be used.
 * param @ key - key for update/insert
 * param @ columnFamily - columnFamily changes
 */
public void apply(PartitionUpdate update, UpdateTransaction indexer, OpOrder.Group opGroup, ReplayPosition replayPosition)
{
    long start = System.nanoTime();
    Memtable mt = data.getMemtableFor(opGroup, replayPosition);
    long timeDelta = mt.put(update, indexer, opGroup);
    DecoratedKey key = update.partitionKey();
    maybeUpdateRowCache(key);
    metric.samplers.get(Sampler.WRITES).addSample(key.getKey(), key.hashCode(), 1);
    metric.writeLatency.addNano(System.nanoTime() - start);
    if(timeDelta < Long.MAX_VALUE)
        metric.colUpdateTimeDeltaHistogram.update(timeDelta);
}
```

It is the call to `metric.writeLatncy.addNano` that appears to be in the output
from `cfhistograms` and that is just a write to the memtable. The comment even
suggests that the time to get the appropriate lock is not included in the
timing.

I don't see much value in the Write figures so I use `cfhisograms` purely for
tuning reads.

## SSTables

The `SSTables` column indicates the number of SSTables that had to be read from
disk to satisfy a read. Unsurprisingly you want this to be as low as possible.
In the above sample 99% of reads were satisfied by accessing 2 SSTables. This is
a good result. If compaction gets behind then this number will rise and your
reads will get slower.

This is a great thing to monitor and if it starts to rise you can be proactive
and look at why nodes are getting being in compaction or if you need to change
your data model.

## Partition Size

This is an estimate of the partition sizes. In the above output it says that 99%
of partitions for this table on this node are under 258 bytes. 

This is useful to monitor to see if your partitions are too large.





