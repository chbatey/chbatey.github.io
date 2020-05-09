---
layout: post
title: 'Tiny docker images for Java with GraalVM native image'
author: Christopher Batey
comments: true
tags:
- docker 
- jvm 
- kubernetes
- graalvm
---

Striving for small Docker images when working with Java. Even with Alpine and a cut down JVM you're still looking
at a 70MiB image. Good compared to a 500MiB JDK image or a 200MiB debian slim image but still large compared to what
native languages can produce.

GraalVM native image promises to improve this situation. With GraalVM native image sizes can be as little as 7Mib for a
Java application.

Here are the steps to get a 7MiB Java docker image.

### Install GraalVM

At easy way to install GraalVM and switch between multiple versions is via the [Java version manager (Jabba)](https://github.com/shyiko/jabba). To install jabba:

`curl -sL https://github.com/shyiko/jabba/raw/master/install.sh | bash && . ~/.jabba/jabba.sh`

Then use jabba to install GraalVM, at the time of writing 20.0.0 is the latest:

`jabba install graalvm@20.0.0`

Then finally select it as your JVM:

`jabba use graalvm@20.0.0`

You can validate that GraalVM is being used via:

```
TODO java -version
```

### Install native image

Current versions of GraalVM don't include native image by default, you need to install it with gu:

`gu install native-image`

Then check you have `native-image` on your path:

```
native-image
```

### Compile a native image

If you're on Linux you can do this on your laptop, otherwise you will have to do it in a VM or in a Docker Container. This is because native image builds a binary for your current platform and we plan to put it in a Linux Docker image so we need it to be a Linux native image.

Compile the following Java application:


```java
public class Main {
   public static void main(String[] args) {
       System.out.println("Hello from GraalVM");
   }
}
```

```
javac Main.java
```

```
TODO native image acommand

```

### Putting it in a Docker container

Let's start with the following `Dockerfile`:

```dockerfile
FROM debian:buster-slim

COPY output /opt/output

CMD ["/opt/output"]
```

Building it gives us a docker image of:

```
TODO docker image size
```

Not bad, let's run it:

```
TODO running the container
```


### Getting it down to 7MiB

The image is still so large as it contains most of Debian. If we compile a static native image then we wont' need anything in our image to run it which will really get the size down.

```
TODO
native-image --static
```

Then change the final image to:

```dockerfile
FROM scratch

COPY output /opt/output

CMD ["/opt/output"]
```

If we build this and check its size, it is now ~7MiB.

```
TODO docker image output
```

{% include jd-course.html %}


