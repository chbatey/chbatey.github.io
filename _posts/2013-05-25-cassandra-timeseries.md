---
layout: post
title: Time series based queries in Cassandra 1.2+ and CQL3
---

Cassandra is fantastic for storing large amounts of data, and as of 1.2/CQL3 working with time series data just got a lot easier. Here is a basic example that stores some kind of posts (e.g blog) that can be queried by username and a time period.

Everything is assuming you have Cassandra 1.2+ installed and are using CQL3 with the cqlsh python client. Here are the exact versions I used for the below example:

```
[cqlsh 3.0.2 | Cassandra 1.2.5 | CQL spec 3.0.0 | Thrift protocol 19.36.0]
```

Creating a keyspace:

```
create keyspace cassandraspike WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor': 1};
```

The syntax has changed for creating keyspaces in CQL for cassandra 1.2 so if this fails check you aren't running against a pre-1.2 version of cassandra.
use cassandraspike;

Then create a posts table:

```
create table posts(username varchar, time timeuuid, post_text varchar, primary key(username, time))
```

We've used a compound primary key that is the username and the time. The time is of type timeuuid which is a new data type in CQL3 that us a type 1 UUID that contains a timestamp so it both stores the time of the post and makes our rows unique.

If you're familier with column families then it might interest you to know that the first column in your primary key becomes the column family (CF) row key. Every subsequent part of the primary key becomes part of the CF column names in that CF row. This has two implications:

* There will only be as many CF rows as there are variations of the first element in your primary key. This can be a problem if this element has a very low cardinality as you can end up with very wide CF rows.
* CF columns are stored in order so the fact that each CF column is prefixed with the time means that your data is stored in the order it happened. Making it very efficient to be queried by time.

Hopefully you are sufficiently convinced that your data is going to be stored in a way that is efficient to query by time. So let's store some data in and query it by user name and time.

To insert we'll make use if the now() function which gives you a timeuuid for the current time:
insert into posts(username, time, post_text) values ('chbatey', now(), 'i am writing something about cassandra');

Selecting this back works but gives you a time which isn't too human read able:

```
select * from posts;

 username | time                                 | post_text
----------+--------------------------------------+----------------------------------------
  chbatey | 59ad61d0-c540-11e2-881e-b9e6057626c4 | i am writing something about cassandra
```


That's where the dateOf function comes in handy which converts a timeuuid into a date:

```
select username, dateOf(time), post_text from posts;

 username | dateOf(time)             | post_text
----------+--------------------------+----------------------------------------
  chbatey | 2013-05-25 14:38:14+0100 | i am writing something about cassandra
```

I've now inserted another post at a different time:

```
select username, dateOf(time), post_text from posts;

username | dateOf(time)             | post_text
----------+--------------------------+----------------------------------------
  chbatey | 2013-05-25 14:38:14+0100 | i am writing something about cassandra
  chbatey | 2013-05-25 14:40:35+0100 | i am writing something about cassandra
```

Now lets say you are only interested in the posts chbatey made at 14:38:

```
select username, dateOf(time), post_text from posts where time >= minTimeuuid('2013-05-25 14:38') and time < minTimeuuid('2013-05-25 14:39') and username = 'chbatey';


 username | dateOf(time)             | post_text
----------+--------------------------+----------------------------------------
  chbatey | 2013-05-25 14:38:14+0100 | i am writing something about cassandra
```

This query is more complicated. It makes use of another cql function: minTimeuuid. The minTimeuuid function gives a fake (as it isn't unique) timeuuid that is the smallest possible timeuuid for the given time. This is very hand when you want to do less than/greater than queries on timeuuid fields.

In the above query we are getting everything that is greater than the minimum timeuuid for the time 2013-05-25 14:38 but less than the minimum timeuuid of the next minute 2013-05-25 14:39.

Rather than that we could have used the very similar function maxTimeuuid function with 2013-05-24 14:39:59. However I find original one easier to understand as I read it as greater than or equal to 14:38:00 and less than 14:39:00.