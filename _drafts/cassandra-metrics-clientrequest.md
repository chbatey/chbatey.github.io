---
layout: post
title: 'Cassandra ClientRequest metrics'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Cassandra records a set of `ClientRequest` metrics that record min, max, mean as
well as a set of percentiles for reads, writes, CAS reads and CAS writes.

You can get these via JMX, nodetool or my preferred method is to export them to systems like
Graphite with the [metrics-report
configuration](http://www.datastax.com/dev/blog/pluggable-metrics-reporting-in-cassandra-2-0-2). 
Opscenter can also be used but given it is no longer compatible for OS Apache
Cassandra I don't use it even with DSE as I 
want a consistent way to manage Cassandra clusters. 

Metrics in Cassandr are prefixed with
`org.apache.cassandra.metrics` which I'll abbreviate to `o.a.c.m`.

The metrics we're interested in are:

```
o.a.c.m.ClientRequest.{Read, Write, CASRead, CASWrite}.Latency.{Mean, Min, Max,
75thPercentile, 99thPercentile, 999thPercentile}
```

You can see these in a JMX viewer such as jconsole:

// Pic of jconsole

You can get access to them via node tool with: `nodetool proxyhistorgrams`

// Nodetool output

When writing this I noticed that the CASRead and CASWrite metrics were missing
so I added them in this jira.

And finally my favourite I push these statisitcs out to Graphite and use
Graphana to display them:

// Pic of Graphana

What do these numbers mean? For that we need to understand a little about
Cassandra's internal architecture. In extremely simplistic terms Cassandra has
three layers: 

* Client APIs: Thrift, Native protocol for CQL
* Dynamo: for handling calling out to other nodes to meet the requested consistency
* Storage: Persisting the node's data on disk

// Picture of client APIs, Dynamo, Storage

The ClientRequest latencies, or proxy stats, are recorded at the Dynamo layer
(called the `StorageProxy` in the C* code in `o.a.c.service` package). So it includes:

* Calls out to other nodes to meet consistency
* Local disk access if necessary
* Syncrhonous read repair????

What's not included:

* Network between client and server
* De-serialising the message
* Serialisation on the way out
* Any GC/OS pauses in the above two

So don't be surprised if the ClientRequest metrics are substantially lower than
what your application perceives. They are just one piece of the puzzle. To get
get the full picture you also need to record:

* Read/Write request metrics from the driver or your own data points before entering the driver
* Full service time from your application's point of view 

That way you'll know where the latency lies.
