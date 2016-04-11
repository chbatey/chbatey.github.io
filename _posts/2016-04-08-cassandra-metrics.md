---
layout: post
title: 'Cassandra metrics'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Cassandra exposes a lot of metrics. Lets have alook at them in a very rough
order of importance:

* [ClientRequest aka proxyhistograms](/cassandra-clientrequest-metrics.html) 
* [ColumnFamily aka cfhistograms](/cassandra-columnfamily-metrics.html)
* [TheadPool aka tpstats](/cassandra-tpstats.html)

For alot of these articles I have show metrics using graphite. For local testing
I tend to spin up a three node cluster using CCM and have each node push metrics
to a grahite instance running with Docker `hopsoft/graphite-statsd`.
