---
layout: post
title: 'Cassandra monitoring'
author: Christopher Batey
comments: true
tags:
- cassandra
---

It is very important with Cassandra that proactive with your monitoring rather
than waiting for applications to start experiencing errors.

## Tools

* Monit
* Nagios + Riemen  +  Graphite + Collectd or..
* Prometheus (all of the above)
* Logging: ELK (Elasticsearch, Logstash and Kibanna)
* MX4J - JMX to HTTP

## Host 

Collecting these stats is typically done with Statsd pushing to Graphite or the
Prometheus Node Exporter pushing to Prometheus.

`Ports` 9042 (Native), 7199 (JMX), 7000 (Inter node)

`Disk space` Alert early. Cassandra requires a lot of free space to do compaction.
Typically 40% warning and 50% critical.

`Disk latency`

`Disk throughput`

`Swap` should be disabled but still alert on it just in case.

`Clock drift` NTP can log out anytime it had to make a large adjustment.

`Ping between nodes` Look for any increased latency between nodes.

`Network` Network saturation. Check repair throttling.

`CPU load average` Warning at greater than cores. Critical at greater than 5 * cores.

### Cassandra Node

Read and Write throughput. Alert on spikes if not expected. Application
mis-behaving. Know what you've capacity planned for any alert on any unexpected
load.

Read and Write latency. Set a threshold and alert on it.

`Pending compactions` Hopefully always 0

`Blocked memtable flush` Indicates disks can't keep up and often a faulty disk.

Where as the blocked memtable flush is a thread pool worth special mentioning
any blocked task is worth alerting on.

`Dropped messages`. This indicates Cassandra is work shedding. Temporary is
okay if your applications aren't seeing timeouts.

`Hinted hand off pending` Should drop down to 0 very quickly.

`SSTables per node` Just to see differences between nodes. No right answer.



### JVM

Garbage collection.

`New gen pause rate`.

`New gen total pause per time period`

`Full GC`

### Table

* `Max partition size`
* `Number of partitions`
* `Number of cells in a partition`

## Things to think about

* Retention of data
