---
layout: post
title: Testing your RESTful services with Groovy, Spock and HTTPBuilder
---

I like to test the HTTP portion of the RESTful web services I build in isolation.
This is because, in the languages I use (Java, Scala), most of the code responsible
for taking in HTTP requests and parsing them is very framework specific and simply
delegates the actual logic to another part of my application.
Ideally I can launch this part of my application in the same way as it is launched
in production but with the rest of the application stubbed out.

Recently I've been using Groovy/Spock for these integration tests,
here's how for a really simple web service. In this example it's been built with Spring Boot.

```java
@Controller
public class ExampleWebAppController {

    @RequestMapping("/exampleendpoint")
    public @ResponseBody Payload exampleEndpoint(@RequestParam(value="input") String input) {
        return new Payload("Something really important: " + input);
    }
}
```

Let's not go into the details of how Spring boot work but it is essentially the same application as in Spring's guide.

The web application has a single path: ```/exampleendpoint``` which takes in a single query param: ```input``` and returns a JSON object with a single field payload with the value: Something really important: with the input appended.

I want to test that the web service takes in HTTP, takes the query parameter out and builds the JSON correctly. With Groovy/Spock/HTTPBuilder here's how it looks:

```groovy

class ExampleWebAppSpecification extends Specification {
    def "Should return 200 & a message with the input appended"() {
        setup:
        def primerEndpoint = new RESTClient( 'http://localhost:8080/' )
        when:
        def resp = primerEndpoint.get([ path: 'exampleendpoint', query : [ input : 'Get a hair cut' ]])
        then:
        with(resp) {
            status == 200
            contentType == "application/json"
        }
        with(resp.data) {
            payload == "Something really important: Get a hair cut"
        }
    }
}
```

Let's look at what is going on here:

* In the setup phase we create a RESTClient
* In the exercising phase we make a call out to the web application.
* In the verification phase we check that the response code is 200, the content type is "application/json" and the body is JSON with a single payload field that contains "Something really important: Get a hair cut"

Benefits over writing this in Groovy/Spock over say just using Java/JUnit:

* The HTTP libraries are easier to use with less code
* The assertions are more expressive
* More magic, for instance the second with() block works on a Groovy map that has automagically created

The above example doesn't deal with how to start and stop the web service under test, I left that out as it is very web framework specific.

What do you need in your build to get this setup? I used Gradle for this example so it is probably easier to just check out the build.gradle, a lot of it can be ignored as it is related to Spring Boot rather than Spock. Here are the important bits:

The plugins:

```groovy
apply plugin: 'spring-boot'
apply plugin: 'java'
apply plugin: 'groovy'
```

And the dependencies:

```groovy
compile("org.codehaus.groovy:groovy-all:2.2.0")
compile("com.fasterxml.jackson.core:jackson-databind")

testCompile("org.spockframework:spock-core:0.7-groovy-2.0")
testCompile("org.codehaus.groovy.modules.http-builder:http-builder:0.7+")
testCompile("net.sf.json-lib:json-lib:2.4+")
```

The entire project is [here](https://github.com/chbatey/spock-web-testing). Happy testing.


