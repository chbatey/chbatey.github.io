---
layout: post
title: 'Cassandra SSTable attached secondary index'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Cassandra 2.1 has secondary indexes, Casandra 3.0 introduced Materialized views.
Now Cassandra 3.4 has a third tool to avoid duplication: SSTable attached
secondary indexes. So when should you use each one? 

_As of writing the 3.* branch isn't production worthy just yet so this is for when
it has stabilised._

The only time you should use a secondary index in Cassandra is when you're also
going to include the partition key. That limits the query to a single node.
Otherwise your query will require most of the cluster as secondary indexes are
partitioned with the base table. They are also extremely limited: only allowing
exact matches.

Materialized views on the other hand do full duplication as you would do in your
application. This is great but quite heavy weight.

SASI fit some where in the middle and will replace every use case I can think of
for secondary indexes. They allow us to:
  
* Equality search
* Inequality e.g lt, gt
* LIKE

They are still partitioned by the base table so executing queries without a
partition key is still a __bad__ idea IMO. But can do some fantastic queries within
a partition that we couldn't do with old SIs.

Here's a table similar to one I had for a new application recently. It stores an order but rather than
doing it state based like you would in a relational database it stores the events like
`ORDER INITIATED`, `PAYMENT RECEIVED`, `ITEMS DISPATCHED`, etc. The current state can be attained
by replaying the event through an in application state machine.


```
CREATE TABLE orders ( id text, 
                      when timeuuid, 
                      event text, 
                      message text, 
                      primary KEY (id, when)) ;
```

This works nicely for looking up an order status and history but the requirement came up to 
audit some types or errors like `PAYMENT FAILED`. This application is going to production ASAP so it
is on Cassandra 2.1 where the solution was a separate table  but when 3.* becomes production worthy we could use a SASI instead:

```
CREATE CUSTOM INDEX ON orders (message) 
  USING 'org.apache.cassandra.index.sasi.SASIIndex';
```

Insert some data:

```
insert into orders (id , when, event, message ) VALUES 
  ( 'OR123', now(), 'INITIATED', 'Customer started order on the website');
insert into orders (id , when, event, message ) VALUES 
  ( 'OR123', now(), 'SYSTEM_CANCELLED', 'Error: Payment failed');
```

Our normal order status query works fine as we just query using the partition key:

```
select * from orders where id = 'OR123';
```

Now imagine we want to find all the events for an order than has "Error" in the
message. With the SASI we can query like this:


```
select id, dateOf(when), event, message from orders where id = 'OR123' 
  AND message LIKE 'Error%' 
```

We're still including a partition key. This isn't mandatory. So we could get Errors for all orders but
this is a huge query as it will have scan the entire token range.

### The details

SASI indexes come in three varieties:
 
* PREFIX - the default. Allows queries like `Error%` but not `%Error` or `%Error%`
* CONTAINS - Allows `%Error%`, `%Error` and `Error%` 
* SPARSE - Like PREFIX but designed to be more efficient for large amounts of numeric data such as timestamps.

Don't be tempted to make all your SASIs CONTAINS. The extra functionality comes at the cost of efficiency
and space.

You pick the type when you create the index:

```
CREATE CUSTOM INDEX orders_message ON orders (message) USING
'org.apache.cassandra.index.sasi.SASIIndex' WITH OPTIONS = { 'mode' : 'CONTAINS'};
```

Under the covers Cassandra is creating a B+ tree based index for each SSTable on
disk that has pointers directly into the SSTable. As Cassandra SSTables are
immutable these never need to be updated in place. When compaction takes place it
builds a new index for the compacted SSTable. Old Cassandra SI went through the 
regular ColumnFamily under the covers so had the full write path to go through.

The Memtable is also indexed so they aren't just SSTable indexes.


### Post extensions

* Create a set of common data models and measure: time take for queries. Disk
  used for the indexes. Memory used for indexes. 

  
