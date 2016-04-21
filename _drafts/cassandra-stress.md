---
layout: post
title: 'Cassandra stress'
author: Christopher Batey
comments: true
tags:
- cassandra
---

Cassandra stress cheat sheet.

```
cassandra-stress (write|read|mixed) 
```

Single thread 

```
cassandra-stress write -rate threads=1
```

Single thread rate limit

```
cassandra-stress write -rate threads=1 limit=10/s
```
