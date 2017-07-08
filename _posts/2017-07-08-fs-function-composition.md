---
layout: post
title: 'Function composition over layers'
author: Christopher Batey
comments: true
tags:
- scala
- fp
- func-scala
---

In mainstream OO you often come across a layered or hexagonal architectures e.g.

* Web/Presentation
* Core domain logic
* Infrastructure e.g. Database access

Where the goal is to keep your core domain logic pure and not tie it to specific databases and web frameworks.

This is admirable goal and one we want to keep when we move to a FP style. 

Let's see an example of a HTTP request that is to result in a person being registered as a customer. 

The step we need are:
* Deserialise the response into a person, we'll do this in the web controller
* Pass the person to an application service
* Generate a customer case class for the person
* Save the new customer to the database
* Return the customer to the web controller
* Serialise the customer to go over the wire

To keep things simple we'll assume our fake web framework gives us the payload as a string.

Here's how it might work in a layered architecture, most of the methods aren't implemented:

```scala
object Layers {
  case class Person(name: String)
  case class Customer(customerNumber: Int, p: Person)

  type HttpRequest = String
  type HttpResponse = String

  trait Serialiser {
    def serialise(c: Customer): HttpResponse
    def deserialie(input: HttpRequest): Person
  }

  class WebController(service: ApplicationService, serialise: Serialiser) {
    def register(p: HttpRequest): HttpResponse = {
      val person = serialise.deserialie(p)
      val c = service.registerCustomer(person)
      serialise.serialise(c)
    }
  }

  class ApplicationService(dataStore: CustomerRepo) {

    def registerCustomer(p: Person): Customer = {
      // validate them?
      // Generate a unique customer number?
      val c = Customer(1, p)
      dataStore.saveCustomer(c)
      c
    }
  }

  trait CustomerRepo {
    def saveCustomer(p: Customer): Unit
  }

}
```

We've modelled the problem with classes and used dependency injection to allow the layers
to communicate with each other.

## How would this look functionally?

Rather than a set of objects that depend on each other our goal is to build a function out of smaller functions! 

In Scala you can take a function from `cat => dog` and a function from `dog => snake` and turn it into a function from `cat
=> snake`! How cool is that? Though please don't call this on any of your pets.

If you think about it a http request is just a function (in our fake web framework these will be just type aliases for
strings):

```scala
HttpRequest => HttpResponse 
```

This time we'll make each stage a function then compose them into our register customer endpoint that'll be of this
type.

If you are the type of developer who refractors code into small methods that are called from a top level method that
reads like prose then you will love function composition.

You can do away with your classes and methods calling methods and replace it with the composition of functions e.g.

```scala
object FunctionalLayers {
  case class Person(name: String)
  case class Customer(customerNumber: Int, p: Person)

  type HttpRequest = String
  type HttpResponse = String
  type WebRequest = HttpRequest => HttpResponse

  val serialiseCustomer: Customer => String = ???
  val deSerialisePerson: String => Person = ???

  def createCustomer(p: Person): Customer = {
    // remove the saving logic
    Customer(1, p)
  }

  def saveCustomer(c: Customer): Customer = {
    // do DB stuff, we'll deal with failure in a later post
    c
  }
  
  val registerCustomer: WebRequest = 
    deSerialisePerson andThen
    createCustomer andThen
    saveCustomer andThen
    serialiseCustomer
}
```

What has changed?
* No more classes, we just have functions. In real code we'd put the functions for each layer in their own module (an
  object in Scala)
* No more functions that depend on other functions. The wiring is done at the end with function composition.

Look at the beauty of our web request now:

```scala
val registerCustomer: WebRequest = 
    deSerialisePerson andThen
    createCustomer andThen
    saveCustomer andThen
    serialiseCustomer
```

This function types to a `WebRequest` which is a type alias for `HttpRequest => HttpResponse`. When we compose functions
like this we end up with a function that takes the input parameters of the first function `deSerialisePerson` and
returns the output of the last function in the composition `serialiseCustomer`. We don't need to test that method X
calls method Y like with the layered approach, the Scala compiler does that for us.

Now of course the DB function could fail and we'll deal with that in a later post. 

The important points to take away:
* Each function can now be tested in isolation, no more mocks.
* You don't need to look in the implementation of one function to see how it calls the next layer. This is all described
  in the function composition
* Function composition is awesome


