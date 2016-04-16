---
layout: post
title: 'Cassandra: Upgrading'
author: Christopher Batey
comments: true
tags:
- cassandra
---
# Upgrading Cassanddra

Bad things can happen when you upgrade a cluster but as long as you follow a few simple rules you can do it without downtime.

[First check out any specific issues for your version](http://docs.datastax.com/en/latest-upgrade/upgrade/cassandra/upgradeCassandra_g.html).

* Never skip a major version. Remember that prior to Cassandra 3 a major version is the first or second number. So 2.0 to 2.1 is a major version change.
* Get to the latest fix release before moving a major version. E.g the highest final number.

There are scenarios where you can break these rules with specific versions but I never do it. The only exception is Cassandra 2.2 which I never intend to put in production.

Then:

* Repair before the upgrade
* Then disable any repairs / make sure that no repairs are in progress
* Remove any process that will modify the schema.
* Do one node at a time:
  * Run `nodetool drain` to stop accepting connections and flush memtables
  * Take a snapshot in case we need to roll back
  * Stop the node
  * Upgrade binary
  * Start the node
  * run `nodetool upgradesstables`
* Once all nodes are done then enable your automated reapir

#### Monitoring
* Hints will build up if writes are coming in: `o.a.c.m.Storage.TotalHints` This is an ever increasing counter so don't expect it to go down but it should stop going up once you bring the new node 
* When you bring the node back up you will see each node's  `o.a.c.m.Storage.TotalHintsInProgress` go up then hopefully drop back down to 0.
* As you're replacing a single node you can also see just that node's hints at each other node with `o.a.c.metrics.HintedHandOffManager.Hints_created-{IP}`. Again this is an ever increasing count.

