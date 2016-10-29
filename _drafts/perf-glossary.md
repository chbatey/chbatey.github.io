--
layout: post
title: 'Performance Glassary'
author: Christopher Batey
comments: true
tags:
- perf
---

IPC - Instructions per cycle
CPI - Cycles per instruction

Modern processors have many ports and can run many instructions in parallel with
a single hardware thread. IPC and its inverse, CPI are a measure of how many
instructions  the CPU achieved in each cycle. This can be less than 0 if the CPU
is waiting (e.g. on memory) or up to ~4 if doing a lot of mathmatical
operations.


