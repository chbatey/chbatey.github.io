---
layout: post
title: 'Functional Scala for everyone'
author: Christopher Batey
comments: true
tags:
- scala
- fp
- func-scala
---

This is the start of a set of blog posts to encourage and educate developers
about functional Scala. 

Whereas a basic understanding of Scala syntax and some programming experience
is expected there is no assumption the reader has any knowledge of FP concepts or Haskell.

The target audience is developers that have moved to, or are considering the move from
OO languages like Java and C# to the wonderful world of Scala.

Topics covered:
- [Functional composition over layers]({{ "/fs-function-composition" | prepend: site.baseurl }})
- [Managing effects in the type system]({{ "/fs-failure" | prepend: site.baseurl }})
- Dependency injection without a framework, and closely related:
- How to manage database connections
- Managing more effects e.g. Future[Either]
- Putting it all together in a real application


