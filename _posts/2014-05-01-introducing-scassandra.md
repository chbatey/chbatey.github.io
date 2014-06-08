---
layout: post
title: Introducing Scassandra - Testing Cassandra with ease
---

[Scassandra](http://www.scassandra.org) (Stubbed Cassandra) is a new open source tool for testing applications that interact with Cassandra.

It is intended to be used for:

* Unit testing DAOs that interact with Cassandra
* Acceptance testing applications that interact with Cassandra

The first release is primarily aimed at Java developers. Subsequent releases will be aimed at people writing black box acceptance tests.

It works by implementing the server side of the CQL binary protocol, meaning that Cassandra drivers, such as the the Datastax Java Driver, believe it to be a real Cassandra.

The original motivation for developing Scassandra was to test edge case / failure scenarios. However it quickly became apparently it is just as useful with regular happy path unit tests.

### So how does it work?

When you start Scassandra it opens two ports:

* Binary port: this is the port you'll configure your application with rather than the binary or a real Cassandra.
* Admin port: this is for Scassandra's REST API that provides priming and retrieving a list of executed queries.

By default, when your application connects to Scassandra and executes queries Scassandra will respond with a result with zero rows.

Then via Scassandra Java Client, and later the REST API, you can prime Scassandra to return rows (where you can specify the column types and values), read request time outs, write request time outs and unavailable exceptions.

After you've run your tests you can verify the queries and prepared statements that your application has executed. Including the consistency the queries have been executed with. So if you have requirements to execute certain writes at a high consistency but other queries can be at a lower consistency this can be tested via black box acceptance tests.

The benefits of Scassandra over testing against a real Cassandra instance:

* Test failures deterministically: where previously you would need to have a multi node cluster with the ability to bring down nodes.
* Test the consistency of queries. This has come up at my workplace where a requirement was that for most queries we can downgrade consistency when there are failures but for certain important writes they had to be executed at QUORUM.
* Have fast running DAO tests that don't require mocking out driver classes or a real Cassandra running.

### So how do I use Scassandra?

The first release of Scassandra is aimed at Java developers. Scassandra comes in two parts:

* Scassandra server. This is a Scala application that has been put in Maven central with a pom that will bring in its transitive dependencies.
* Scassandra Java client. A thin wrapper written in Java to make using Scassandra from Java tests easy. This has methods to start/stop Scassandra and classes that prime / retrieve the list of executed queries.

For the first release it is expected that Scassandra will only be used via the Java client and no one will use it as a standalone executeable or interact with the REST API directly.

To get started with the Scassandra Java client then go [here](http://www.scassandra.org/java-client). Or checkout the example project [here](https://github.com/chbatey/scassandra-example-java).

If you aren't using Java or a language that can easily use the Java client then the next release will be for you where we'll build a standalone executable and from that release on we'll make the REST API backward compatible as we'll expect people to use it directly.

It is all open source on github and you can find Scasandra server [here](https://github.com/scassandra/scassandra-server).

The Java client is [here](http://www.scassandra.org/java-client).

And all the details of the REST API e.g how to prime are on the Scassandra sever website [here](http://www.scassandra.org/).