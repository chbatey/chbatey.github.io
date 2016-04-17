---
layout: post
title: 'What to do when Cassandra burns'
author: Christopher Batey
comments: true
tags:
- cassandra
---

A typical Cassandra installation is many nodes possibly spread across many DCs.
What do you need to do in various failure scenarios:

* Node down for less than the hint window
* Node down for greater than hint window but less than gc_grace_seconds
* Node down for longer than a gc_grace seconds

### Less than hint window

Restart the node.

(Optional) Check for pending hints for that node in each other node.

* Hints will build up if writes are coming in: `o.a.c.m.Storage.TotalHints` This is
an ever increasing counter so don’t expect it to go down but it should stop
going up once you bring the node back up
* When you bring the node back up you will see each node’s
`o.a.c.m.Storage.TotalHintsInProgress` go up then hopefully drop back down to 0.

### Greater than hint window

Hints won't reapir the data now. If it is frequently read data and you have read
repair on then it will slowly fix its self but you probably want to repair the
node after bringing bit back up.

### Greater than gc_grace seconds

Deletes for data that is on the downed node will have been cleaned up by
compaction. So bring the node backup is very dangerous. Replace the node.

### Todo

* Endless compactions
* Out of disk space
* Ever growing partition
* Tombstoness


