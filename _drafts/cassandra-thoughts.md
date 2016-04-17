---
layout: post
title: 'Cassandra thoughts and questions'
author: Christopher Batey
comments: true
tags:
- cassandra
---

### Hints


### Repair

* Why can't you bring back a node that has been down longer than
  gc_grace_seconds? _Tombstones for data that is on the downed node will have
  been garbage collected so bring the node back will re-introduce old data_

* If a node is down for longer than gc_grace_periods but you don't do any
  deletes can you bring it back and repair it?


### Vnodes

* Can you have vnodes in one DC and not the other? _Yes this is the recommended
  way to upgrade to vnodes.
