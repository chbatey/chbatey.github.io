---
layout: post
title: Unit Testing Cassandra Applications
---

I often get asked what is the best method for unit and and integration testing Java code that communicates with Cassandra.

Here are what I think the options are:

* Mocking libraries: Use mocking libraries such as Mockito to mock out the driver you are using
* Cassandra Unit
* Integration test the class in question with against a real Cassandra instance running locally
* Stubbed Cassandra (disclaimer: I made this)

Lets take these one by one and look at their respective advantages and disadvantages. The things to consider are:

* Speed of the tests
* Able to run the tests concurrently and how easy this is to achieve
* Able to test everything? Even failures?
* Are the tests brittle?
* Readability of the tests
* Requirements on environment e.g do you need a Cassandra instance installed?
* Confidence it will work against a real Cassandra
* Subjective nonsense by writer of this blog

## Mock the library

You can use a combination of factories and mocking to stop the driver actually interacting with Cassandra. Then verify your code's interactions with these mocks.

Advantages:

* Fast - no I/O
* Execute tests concurrently
* No Cassandra instance required on machine your code compile runs
* Can test everything including the various failures as you can mock the library to throw ReadTimeout exceptions etc

Disadvantages:

* You may mock the driver to behave in a different way to how it will behave
* Very hard to understand tests due to large amount of boiler plate mocking code
* Very brittle tests. Change your driver, change your tests! Change from a query to a prepared statement, change your tests!
* A lot of boiler plate. Take the Datstax driver for example, it returns a ResultSet, which is iterable, fancy writing the code that mocks it returning many results?

Conclusion:

* Don't do it if you wish to remain sane

## Cassandra Unit

Cassandra Unit is a tool for starting an embedded Cassandra in the JVM your tests are running. It also has a great API for ingesting data into Cassandra for your tests.

Advantages:

* Pretty fast. It is all in process.
* Can run tests concurrently if they use different keyspaces and none of the tests turn it off etc
* Can use CQL to load data as it is a real Cassandra. I think this leads to readable tests
* No Cassandra required on machine. Can use the it via a Maven dependency
* High confidence your code will work against a real Cassandra

Disadvantages:

* Unable to test failure scenarios. What is a read time out when there is only one node running in the same JVM as your test?
* No way to verify consistency of queries

Conclusion:
* Very useful tool for in process happy path tests


## Integration style tests using a real Cassandra

This is something I've done a lot of recently. Every dev machine at my current work place has a Cassandra instance running (using the awesome tool ccm). Then we test our DAOs by assuming it is there and doing testing in a dynamic keyspace.

Advantages:

* Can run tests concurrently if they use different keyspaces and none of the tests turn it off etc
* Tests aren't brittle, can change from queries to prepared statements, or the queries involved without changing tests
* Readable tests - all data setup is done in CQL
* Very high confidence it will work against a real Cassandra as the test is against a  real Cassandra :)

Disadvantages:

* Probably the slowest option. But it is still millisecond quick.
* Hard to test scenarios other than turning the node off. This then makes the tests slow.
* Cassandra required to build and run tests

Conclusion:

* Slightly slower but very useful for testing happy paths
* Very similar to using Cassandra unit


## Stubbed Cassandra

Stubbed Cassandra is a new open source tool that pretends to be Cassandra and can be primed to returns rows, read timeouts, write timeouts and unavailable errors. It can be used via a maven dependency or as a standalone server.

Advantages:

* Very fast
* Can test all types of errors and be confident in what the driver does as the driver thinks it is a real Cassandra
* Can run many instances inside the same JVM listening on different binary ports. So can run tests concurrently with no extra effort e.g no requirement to use different keyspaces
* Tests less brittle than mocking the driver. Can change driver without changing test but if you change queries you need to update your priming
* No requirement to have a real Cassandra. Just brought in by a maven dependency

Disadvantages:

* Slightly more brittle than a real Cassandra/Cassandra unit. Requires priming on the query, priming for each prepared statement
* Slightly less confidence it will work against a real Cassandra as it isn't a real Cassandra. But more confidence than mocking
* It is new and does not support all of Cassandra's features, so if you use a feature that Scassandra doesn't support you are stuck!

Conclusion:

* Best solution for all error case testing
* Best solution if you need to execute tests concurrently