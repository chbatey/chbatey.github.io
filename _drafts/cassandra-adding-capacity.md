---
layout: post
title: 'Cassandra: Adding capacity'
author: Christopher Batey
comments: true
tags:
- cassandra
---


### Adding nodes

If you're not already using vnodes migrate to them.

Then follow the [DataStax
docs](http://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_add_node_to_cluster_t.html).

### Switching out a node's hardware

This is particularly popular on cloud providers when you decide to change the
instance type you are using.

If using ephemoral disks just follow the steps that for replacing a dead node.
However if you can copy the data to the node and are definitely not changing the
cluster topology and the new hardware will have the same IP you can do this then run a repair to get the data that you
missed.

