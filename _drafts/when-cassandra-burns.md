---
layout: post
title: 'What to do when Cassandra burns'
author: Christopher Batey
comments: true
tags:
- cassandra
---

A typical Cassandra installation is many nodes possibly spread across many DCs.
What do you need to do in various failure scenarios:

* Node down for less than the hint window
* Node down for greater than hint window but less than gc_grace_seconds
* Node down for longer than a gc_grace seconds

## Less than hint window

## Greater than hint window

## Greater than gc_grace seconds




