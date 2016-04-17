---
layout: post
title: 'Cassandra: Adding a DC'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Reasons to add a new DC to an existing Cassandra cluster are varied.

* Redundancy. Handing a DC failure.
* Moving DC with no down time. Once the move is complete decommission the old
  DC.
* Similar to the above. Changing cloud provider or moving from a cloud provider
  to on premises.
* As an upgrade strategy.

Regardless of the reasons the procedure is the same. The [DataStax
docs](https://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_add_dc_to_cluster_t.html) are good on
this topic. Read and follow those. The points I see missed the most:

* Ignoring the docs and doing one node at a time
* Application consistency. Must move to LOCAL_ consistencies
* Using a DC aware load balancing policy in your client
* Not repairing after the rebuild. As each token range is streamed from a single
  source, that source could have been out of date.

### Monitoring 

The rebuild is a streaming operation that will be visible via `nodetool netstats`




