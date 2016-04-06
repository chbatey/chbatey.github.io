--
layout: post
title: 'Docker vs the JVM: Overview'
author: Christopher Batey
comments: true
tags:
- docker
---

Over the last year I've been working on a large project involving a set of services
written primarily in Java, packaged in Docker containers, and
deployed to a set of Kubernetes clusters.

Where as there is a lot of hype around Docker, especially at JVM conferences
I've attended such as Devoxx, Geecon, Javaland, I've found running the JVM inside
a Docker container isn't as easy as I'd have expected. So follows is a set of
posts with my lessons learned and all the stuff I wish I had known before I
started the project. A lot of it is applicable even if you aren't using Docker.
CGroups for instance, knowing a bit about how these work will help you out on
any Systemd based Linux distribution.

* Java base images
* CGroups: Memory
* Groups: CPU
* File systems
* Process spacew
