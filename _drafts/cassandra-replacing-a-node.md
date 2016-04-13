---
layout: post
title: 'Cassandra: Replacing nodes'
author: Christopher Batey
comments: true
tags:
- cassandra
---

# Replacing a node (Dead or Scale up)

There are two scenarios:

* The node is still alive
* The node is dead

These procedures change version to version. This is for Cassandra 2.1. One thing to note is you can't bootstrap seed nodes. 
So if the node is a seed node first remove it from all other nodes seed lists and appoint a new seed.

### Replacing a node

There are many reasons to replace a node that is not dead. 
EOL hardware, changing hardware strategy for cost / performance reasons in AWS, etc.

Don't try and upgrade at the same time as changing hardware. 
You don't want a multi verson cluster for any longer than you have to as you need to turn off streaming.
One thing at a time! Don't try and be too clever.

[Check out the latest docs for your version](https://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_replace_live_node.html).

Assuming you are running with VNodes you should add the node then decommission the old node. 
The DS docs talk about using Host ID. This is only required if using `nodetool removenode`. 
If the node is alive you can use `nodetool decommission` on the actual node.

This allows you to add the new node before you remove the old node. So no loss in capacity. 
If the node is dead then obviously this isn't the case.

#### Effects
* Unless you're using `ALL` you should be fine.

### Replacing a dead node

Rather than bootsrapping and force removing it is better to start a new node with 
`-Dcassandra.replace_address=[dead_node]`. 
This means the new node will take over the same token ranges as the old node and no unncessasry data movement is required.


