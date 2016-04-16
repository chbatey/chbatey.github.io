---
layout: post
title: 'Cassandra materialized views'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Cassandra materialised views (MVs) were introduced in Cassandra 3.0. I've
[written
about them before](/cassandra-30-materialised-views-in.html) they were released and discovered their semantics by a
combination of trial and error, reading the
[JIRA](https://issues.apache.org/jira/browse/CASSANDRA-6477) and talking on the #cassandra
channel on freenode. 

This post is just to summerise what was actually released and assumed you've read the
[previous one](/cassandra-30-materialised-views-in.html).

http://www.datastax.com/dev/blog/new-in-cassandra-3-0-materialized-views
http://www.datastax.com/dev/blog/understanding-materialized-views
https://docs.google.com/document/d/1sK96wsE3uwFqzrLQju_spya6rOTxojOKR9N7W-rPwdw/edit
https://issues.apache.org/jira/browse/CASSANDRA-11069?jql=labels%20%3D%20materializedviews

### Metrics

### Building a view

### Repair

* No read repair between on the view. Reads in the view only read repair data
  already in the view on different nodes
* There is read repair on the base to the view 
  normal writes

### Interesting points

* Writes the base with a view happen sequentially which is not the case for


### Tips for using

* Don't include columns you don't need in your view tables. One it wastes space
  as it is full duplication. Secondly if a write to a base table doesn't affect
  any views it goes through the normal (fast!) write path. Rather than the two
  batch log / read before write MV write path.


