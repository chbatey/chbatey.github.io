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
o.a.c.m.ColumnFamily.[keyspace name].[table name].{WriteLatency, ReadLatency, SSTablesPerReadHistogram}
```

Remember the simplified three layers of Cassandra:

* Client APIs (Thrift, Native protocol for CQL)
* Dynamo 
* Storage

The ColumnFamily metrics are in the Storage layer in the `o.a.c.db` pacakge. They do not include any network time associated with meeting your consistency level.

The output from node tool looks similar to `proxyhistograms` but also includes two additional COlumns `SSTables` and `Cell Count`

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

The SSTables column indicates the number of SSTables that had to be read from
disk to satisfy a read. Unsurprisingly you want this to be as low as possible.
In the above sample 99% of reads were satisfied by accessing 2 SSTables. This is
a good result. If compaction gets behind then this number will rise and your
reads will get slower.

// TODO turn off autocompaction 


