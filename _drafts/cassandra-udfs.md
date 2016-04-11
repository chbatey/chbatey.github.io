---
layout: post
title: 'Cassandra User Defined Functions'
author: Christopher Batey
comments: true
tags:
- cassandra
---

As of version 2.2 Cassandra supports user defined functions. I [wrote about
these a while
back](/cassandra-aggregates-min-max-avg-group.html) before the were released. 
This post has the released syntax and explores when I would use them in real CQL data modelling.

__TL;DR__ I wouldn't really use them unless I was using User Defined Aggregatres and would wait for Cassandra 3.* to become stable.
UDAs will become a game changer for Cassandra as more functionality like
`GROUP_BY` will be possible within a partition.

### The details

They can be written in any with JSR-223 support. So far that includes: Java, JavaSript, Python, Scala, Ruby.

In a nutshell:

 * The function has to be stateless
 * Can be used in select clauses
 * Can take in any number of columns
 * Have to return a CQL type
 * They are always executed on the coordinator

As they can only transform fields I feel like any code I might be tempted to put in a UDF would be better suited in my application. 
Perhaps an advantage is many applications can make use of the function regardless of their implementation language, but I don't like DB integration so I don't see it.

The one time when they could be useful is when you can reduce the data brought back from the server to your application. 
As it stands Cassandra collections and User Defined Types are always sent in their entirity. 
If you can turn a large UDT or collection into a small value for sending over the wire then you start to gain some benefit.

```
CREATE TABLE Games ( name text, 
                     when date, 
                     scores set<int>, 
                     primary KEY (name));
```

Set total:

```
CREATE OR REPLACE FUNCTION setTotal(input set<int>) RETURNS NULL ON NULL INPUT RETURNS int LANGUAGE java AS '';
```

Set max:

```
TODO
```


Set min:

```
TODO
```

As you can see they can be tedius to write and we have only defined them for
sets of int. There is no way to write a generic function that would work with
Sets of different types.

If I ended up with a UDT or Collection that justified using a UDF to filter it
before sending back to my application I would first consider if I had the
right data model before breaking out the UDFs.


### Under the covers

#### Java UDFs

Your code of your function is inserted into a template called
`JavaSourceUDF.txt`:

```
package #package_name#;

import java.nio.ByteBuffer;
import java.util.*;

import org.apache.cassandra.cql3.functions.JavaUDF;

import com.datastax.driver.core.TypeCodec;

public final class #class_name# extends JavaUDF
{
    public #class_name#(TypeCodec<Object> returnCodec, TypeCodec<Object>[] argCodecs)
    {
        super(returnCodec, argCodecs);
    }

    protected ByteBuffer executeImpl(int protocolVersion, List<ByteBuffer> params)
    {
        #return_type# result = #execute_internal_name#(
#arguments#
        );
        return super.decompose(protocolVersion, result);
    }

    private #return_type# #execute_internal_name#(#argument_list#)
    {
#body#
    }
}
```

And then compiled with the eclipse compiler. This all happens in
`o.a.c.cql3.functions.JavaBasedUDFunction` class. This 
#### Other languages

All other languages are...
