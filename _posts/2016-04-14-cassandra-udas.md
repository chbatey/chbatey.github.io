---
layout: post
title: 'Cassandra User Defined Aggregates'
author: Christopher Batey
comments: true
tags:
- cassandra
---

I previously posted about [Cassandra UDFs](/cassandra-udfs.html). It is worth
reading that post before carrying on as UDAs use UDFs. I've also posted about
UDAs [here](/cassandra-aggregates-min-max-avg-group.html) and
[here](/a-few-more-cassandra-aggregates.html) however these were
before they were released and the syntax and the semantics have since
changed.


###  What aggregates in Cassandra??

Cassandra is designed to scale via all queries being satisfied by a sequential
read of a single partition. Aggregates break that model. So why have they been
introduced?

### Built in aggregates

First lets see exactly what Cassandra is giving us. First there are some built
in aggregates:

* avg 
* countRows
* max
* min
* sum

You can take a look at them in `o.a.c.cql3.functions.AggregateFcts`.

Example usage:

```
CREATE TABLE examples.scores (
    id text,
    when timeuuid,
    player text,
    score int,
    PRIMARY KEY (id, when)
)
```

Then to use the max:

```
select max(score) FROM examples.scores ;

 system.max(score)
-------------------
               105

(1 rows)

Warnings :
Aggregation query used without partition key
```

What is this warning? We just did an aggregation over an entire table. This will only
work for small tables on small clusters. It isn't a scalable query. We want to restrict
our usage of aggregates to within a partition e.g.


```
 select max(score) FROM examples.scores where id = 'GAME1';

 system.max(score)
-------------------
               105

(1 rows)

```

### Custom aggregates

Custom aggregates allow you to reduce a query that returns many rows into a
single value.


To do this we need to:

* Define a function that will be executed, sequentially, for every row, passing
  some state along. 
* Initial state for the first row
* An optional function that will take in the final state and produce a value
  

This sounds very complicated, If you're used to functional languages this can be thought
of as a `foldl` with the optional function allowing a final transformation to a
different type.

In CQL this looks like:

State function:

```
CREATE OR REPLACE FUNCTION avgState ( state tuple<int,bigint>, val int ) CALLED
ON NULL INPUT RETURNS tuple<int,bigint> LANGUAGE java AS '
  if (val != null) { 
    state.setInt(0, state.getInt(0)+1); 
    state.setLong(1,state.getLong(1)+val.intValue()); 
  } 
  
  return state;' 
```

Final function:

```
CREATE OR REPLACE FUNCTION avgFinal ( state tuple<int,bigint> ) CALLED ON
NULL INPUT RETURNS double LANGUAGE java AS '
  double r = 0; 
  if (state.getInt(0) == 0) 
    return null; 
  r = state.getLong(1);
  r /= state.getInt(0); 
  return Double.valueOf(r);'
```

  Creating the aggregate:

```
CREATE AGGREGATE IF NOT EXISTS average ( int ) 
  SFUNC avgState 
  STYPE tuple<int,bigint> 
  FINALFUNC avgFinal 
  INITCOND (0,0);
```

Of course average already exists so this was a waste of time. The first function
is executed once per row passing a `tuple<int, bigint>` along. The initial value
of the state is defined in the aggregate as `INITCOND (0,0)`. Once all of the
rows have been processed the final function is executed which converts the state
of `tuple<int, bigint>` into the final value of type `double`.

### Restrictions

The code you can write inside a UDF and thus either the state or final function
of a UDA is quite limited. 

* It has to be a pure function so no IO, logging, external access of any kind.
* Can't start threads
* No dependencies

### Under the covers

Understanding what is going on under the covers will hopefully give you a good
idea of when you should use UDAs.

When a query contains a UDA it is still executed as normal:

* The coordinator picks a replica to get the data from + digests to meet your
  consistency
* When the rows arrive at the coordinator they are passed sequentially through
  the state function

So the UDA state function is never executed on nodes other than the coordinator. 
This isn't map reduce hence why you shouldn't use UDAs without specifying a
partition key. If you do the entire table will need to be transferred  to the coordinator,
this is likely to put extreme pressure on the coordinator / cause OOMs.

How the state function and final function work is described in a [previous
post](/cassandra-udfs.html). The `o.a.c.cql3.functions.UDAggregate` brings these
together in a simple interface:


```
interface Aggregate
{
    public void addInput(int protocolVersion, List<ByteBuffer> values) throws InvalidRequestException;
    public ByteBuffer compute(int protocolVersion) throws InvalidRequestException;
}

```

The implementation of addInput calls your state function and the implementation
of compute calls your optional final function. Everything is a ByteBuffer as that is how
Cassandra stores values internally. 

