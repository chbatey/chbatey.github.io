---
layout: post
title: 'Cassandra User Defined Types'
author: Christopher Batey
comments: true
tags:
- cassandra
---

User defined types were introduced in Cassandra 2.1. 

Rather than have a table like:

```
CREATE TABLE orders(id text, 
                    action text, 
                    when timeuuid, 
                    item_name text,
                    item_price int, 
                    user_name text, 
                    user_email text, 
                    PRIMARY KEY (id, when, action))
```

We can create types for item and user as follows:

```
CREATE TYPE item (name text, price int) ;   
CREATE TYPE user (name text, email text) ;
```

Then we can create the table as follows:

```
CREATE TABLE orders2(id text, 
                     when timeuuid, 
                     action text, 
                     item frozen<item>,
                     user frozen<user>, 
                     PRIMARY KEY (id, when, action)) 
```

Is it any better? Well we get to stop prefixing things with `user_` and `item_`
so I think it better documents our data model.

You create UDTs for insertion as follows:

```
INSERT INTO orders2 (id, when , action , item, user) VALUES ( 
  'OR1234', 
  now(),
  'START', 
  { name: 'crisps', price: 123 }, 
  { name: 'Chris', email:'chris@internet.com'} ); 
```

And they look like this when you select them:

```
id     | when                                 | action | item                        | user
--------+--------------------------------------+--------+------------------------------+----------------------------------------------
OR1234 | eb257e80-ffb3-11e5-ba92-39784dd51c6a |  START | {name: 'crisps',price: 123} | {name: 'Chris', email: 'chris@internet.com'}

```


How you do it in code is driver specific. I would probably only use them if the
driver I was using supported mapping them to an object as otherwise they can be
awkward to construct.

## Frozen 

As of Casandra 2.1 you have to make the type frozen<yourtype> which indicates
that you can't modify it, you can only replace it in its entirity. 

This can make data modelling with UDTs hard as if an update comes into your
application for just the user email without the user name for an order then we'
first have to read the user UDT to get the user name and then write it back.
This is a major Cassandra anti-patten.

For this reason my rule for UDTs is to only use them when you will always update
the UDT in its entirity.

## User defined types under the covers

A feature I use even less than UDTs is tuples. Yes Cassandra supports tuples,
who knew? UDTs are basically tuples serialised as a blob under the covers. That is why you need
to update the entire UDT at once which makes
data modelling with them somewhat hard for a lot of scenarios. 

For full context check out the [original Jira -5590](https://issues.apache.org/jira/browse/CASSANDRA-5590). 
The removal of the frozen / blob sematics is coming in Casasndra 3.6: [Allow updating
individual subfields of UDT](https://issues.apache.org/jira/browse/CASSANDRA-7423). 

Until then UDTs will be a rarely used feature for me.




