---
layout: post
title: 'GraalVM native image multistage Docker builds'
author: Christopher Batey
comments: true
tags:
- docker 
- jvm 
- kubernetes
- graalvm
---

GraalVM native image can be used to create tiny Docker images as described [here](graalvm-docker-small-images.html).

Here's how to use a multistage Docker build to build the image and produce a tiny image.
First i'll assume that your application has already been built and is avaialble in the Docker
context as app.jar

```dockerfile
TODO add the build
```

The first stage of the build creates the static native image. We use a static image so that
we can use the `scratch` base image for the final image.

The second stage starts from `scratch` and all it adds is the native image built in the 



