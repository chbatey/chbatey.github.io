# Cassandra stress

Cassandra stress is a great tool but is far from intuitive to use.

Here's the basics I use when load testing a newly provisioned cluster and
when I am validating a new schema.

## All the options

It can be hard to use without looking at the source code but you can get quite
far by learning to use the command line help system.


`cassandra-stress help [command]`

Or

`cassandra-stress help [option]`

E.g for command

```
./cassandra-stress help user                                                                                                                          

Usage: user profile=? ops(?) [clustering=DIST(?)] [err<?] [n>?] [n<?] [no-warmup] [truncate=?] [cl=?]
 OR 
Usage: user profile=? n=? ops(?) [clustering=DIST(?)] [no-warmup] [truncate=?] [cl=?]
 OR 
Usage: user profile=? duration=? ops(?) [clustering=DIST(?)] [no-warmup] [truncate=?] [cl=?]

  profile=?                                Specify the path to a yaml cql3 profile
  ops(null): Specify the ratios for inserts/queries to perform; e.g. ops(insert=2,<query1>=1) will perform 2 inserts for each query1
      null
  [clustering=DIST(?)]: Distribution clustering runs of operations of the same kind
      EXP(min..max)                        An exponential distribution over the range [min..max]
      EXTREME(min..max,shape)              An extreme value (Weibull) distribution over the range [min..max]
      QEXTREME(min..max,shape,quantas)     An extreme value, split into quantas, within which the chance of selection is uniform
      GAUSSIAN(min..max,stdvrng)           A gaussian/normal distribution, where mean=(min+max)/2, and stdev is (mean-min)/stdvrng
      GAUSSIAN(min..max,mean,stdev)        A gaussian/normal distribution, with explicitly defined mean and stdev
      UNIFORM(min..max)                    A uniform distribution over the range [min, max]
      FIXED(val)                           A fixed distribution, always returning the same value
      Preceding the name with ~ will invert the distribution, e.g. ~exp(1..10) will yield 10 most, instead of least, often
      Aliases: extr, qextr, gauss, normal, norm, weibull
  err<? (default=0.02)                     Run until the standard error of the mean is below this fraction
  n>? (default=30)                         Run at least this many iterations before accepting uncertainty convergence
  n<? (default=200)                        Run at most this many iterations before accepting uncertainty convergence
  no-warmup                                Do not warmup the process
  truncate=? (default=never)               Truncate the table: never, before performing any work, or before each iteration
  cl=? (default=LOCAL_ONE)                 Consistency level to use
  n=?                                      Number of operations to perform
  duration=?                               Time to run in (in seconds, minutes or hours) 
```


E.g for an option

```
./cassandra-stress help -node                                                                                                                         

Usage: -node [whitelist] [file=?] []

  whitelist                                Limit communications to the provided
  nodes
    file=?                                   Node file (one per line)
       (default=localhost)                     comma delimited list of nodes

```

### Downgrading the protocol

It is common to want to use the shiniest new cassandra stress but you want to
run against an older cluster. Stress now supports this via:

```
./cassandra-stress write -node $CHOST -mode native cql3 protocolVersion=3 
```

The protocolVersion refers to the CQL native protocol version and you can do a
quick google search to find the right one for your version of Cassandra or just
keep going back one until it work.

### Authentication

Your cluster has authentication enabled right? You'll need to add these options:

```
./cassandra-stress write -node $CHOST -mode native cql3 user=$USERNAME password=$PASSWORD
``` 

### Keyspace setup

Unfortunately cassandra-stress creates a keyspace with the `SimpleStrategy` and
a replication strategy of 1. Unless you are using the wrong database this
probably isn't realistic for what you plan to do in production.


```
/cassandra-stress write -node $CHOST -mode native cql3 protocolVersion=3 -schema "replication(strategy=NetworkTopologyStrategy,$DCNAME=3)"  
```

*Gotcha* If you have run cassandra-stress previously then this won't replace the
old keyspace :( 

## Basic write and read test

Finally we can run against a real cluster. I tend to start with a write test,
then read test then a mixture before I go onto custom schemas:

```
./cassandra-stress write -node $CHOST -mode native cql3 user=cirrus_admin
password=$PASSWORD protocolVersion=3

./cassandra-stress read -node $CHOST -mode native cql3 user=cirrus_admin
password=c1rrus protocolVersion=3

./cassandra-stress mixed -node $CHOST -mode native cql3 user=cirrus_admin
password=c1rrus protocolVersion=3

```

Without overriding the `-rate` parameter cassandra-stress is running in `auto`
mode. In this mode the number of threads is increased by 1.5 assuming that the
last increase resulted in an increase in throughput. This continues until a max
thorughput is achieved.

This will fully saturate your clsuter assuming that your cluster is the
bottleneck.

After this test I tend to run some throughput/throttle/fixed (depends on which
version of cassandra stress you are using) to then see what a sustainable
throughput for the cluster is.


## User tests

The user mode os by far the most useful as you can control the schema and type
of load.


## Misc
 
### Running from trunk

I tend to run cassandra-test from trunk. You will need a JDK installed and ant.

Clone the repo from github: https://github.com/apache/cassandra 
Change into the directory then run ant to build cassandra-stress (as well as
cassandra). Build dependencies are detailed on the project's contributing guide:
http://wiki.apache.org/cassandra/HowToContribute

Once it is built then you can run it from `./tools/bin/cassandra-stress`
