---
layout: post
title: 'How-To: Flight recorder'
author: Christopher Batey
comments: true
tags:
- jvm
- perf
---

Use Oracle's JVM rather than an OpenJDK.

To allow flight recorder to be used:

```
-XX:+UnlockCommercialFeatures -XX:+FlightRecorder
```

To start by default use soemthing like:

```
-XX:StartFlightRecording=duration=60s,filename=myrecording.jfr
```

Or attach JMC and do it, or use JCmd.

For using perf:

```
-XX:+DebugNonSafepoints
``

