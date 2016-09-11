---
layout: post
title: "You can't average percentiles"
author: Christopher Batey
comments: true
tags:
- stats
---

One of the first things I check when looking at a new dashboard is for the
dreaded graph that is an average of a percentiles. 

Here is a [brilliant talk](https://www.youtube.com/watch?v=lJ8ydIuPFeU) on the
subject by Azul's CTO Gil Tene.

This post  is my attempt to make why it doesn't
work intuative to developers.

This post is applicable to any setup where
you do something along these lines: 

* Record the duration of something
* Report aggregated statistics (e.g. mean or 99th-ile) to a metrics database 
* Aggregate the aggregates again in the metrics database (e.g. average
  percentiles)

## Recording a duration

Let's assume we have a variable representing the response time of some
application which takes the following 10 values (milliseconds): 


```
11
12
13
14
15
16
17
18
19
20
```

In this sample 90% of the requests took 19ms or less. 

Percentiles are useful as you can see what proportion of your users
have what kind of experience.

Often the raw data isn't sent to the metrics database; instead the database only
stores the aggregate (in this case the aggregate is the percentile):

```
90%-ile = 19
```

Great, we can graph this over time and alert when the %-ile gets above a
threshold. 

All good so far.

We have a second instance of the same application with the following response
times:

```
91
92
93
94
95
96
97
98
99
100
```

These are bit slower for whatever reason. For this instance 90% of the response times are 99ms or less.

Off to metrics database it goes:

```
99%-ile = 99
```

We then average them in metrics database:

```
99 + 19 / 2 = 59
```

Urgh 59? What is this made up number we are now graphing? Are 90% of customers
getting a response time of less than 59? Hell no. Half of them are getting a
much higher response time.

In reality 90% of our customers are getting a response time of 98 or less.

So don't average percentiles.

Another gotcha is when aggregates are made from samples of different sizes.
So you try and average an average from two instances and they received a
different number of requests.

## What can we do?

To aggregate data across instances we need the raw data or at least a histogram
with an appropriate level of precision. I don't think it matters what tool you
use but the one I use at the time of writing and that supports this is
[Prometheus](https://prometheus.io/docs/practices/histograms/).
