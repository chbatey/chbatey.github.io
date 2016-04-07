---
layout: post
title: 'Cassandra command line tools'
author: Christopher Batey
comments: true
tags:
- cassandra
---

The cassandra bin directory is full of useful tools:

### nodetool

This is the go to tool for nearly all cassandra admin and getting metrics for a
single node.

### cqlsh / cassandra-cli

Cqlsh is for executing queries against Cassandra. Cassandra-cli is the old thrift
client.

### sstabkekeys

Lists the keys in a SSTable. Make sure you flush first. I have never used this.

### sstableloader

The Cassandra bulk loader. For bulk loading also see Brian Hess's
[loader](https://github.com/brianmhess/cassandra-loader) that uses the Java
Driver directly.

### sstableupgrade

Offline version of `nodetool sstableupgrade`

### stablescrub

Offline version of `nodetool sstablescrub`

### cassandra-stress

Great utility for stress testing a cluster either with your schema or a auto
generated one. Always useful to smoke test a new cluster with this.

### sstable2json / json2sstable

Does what it says. However it has been removed in Cassandra 3.0 with the new
storage engine so don't build anything that relies on it.

### token-generator

Can generate token values for each node for bootstrapping a new cluster.  Not
required if you are using vnodes.



