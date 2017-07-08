---
layout: post
title: 'Dealing with failures'
author: Christopher Batey
comments: true
tags:
- scala
- fp
- func-scala
---

If you haven't read the [first article]({{ "/fs-function-composition" | prepend: site.baseurl }}) in the series I suggest you do so as this builds on the same example.

We ended the last article with the following code:

```scala
  case class Person(name: String)
  case class Customer(customerNumber: Int, p: Person)

  type HttpRequest = String
  type HttpResponse = String
  type WebRequest = HttpRequest => HttpResponse

  val serialiseCustomer: Customer => HttpResponse = ???
  val deSerialisePerson: HttpRequest => Person = ???

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
```

Next we need to deal with failures e.g. The database down or invalid user input.

Let's start with a new requirement:

A user must specify an email and we're going to validate it at least has an @ sign in. Our beautiful composition now has
to fork or end early in some way. Are we going to revert to imperative programming with if statements? Never!

One of the goals in functional Scala is to represent as much as possible in the type system. This is brilliant
documentation to the next developer to touch the code. Let's see one way to represent failure in the type system.

```scala
  sealed trait Error
  case object InvalidEmail extends Error
  case object DatabaseDown extends Error

  type HttpRequest = String
  type OkHttpResponse = String
  type WebRequest = HttpRequest => Either[Error, OkHttpResponse]

  def validatePerson(p: Person): Either[Error, Person] =
    if (p.email.contains("@"))
      Right(p)
    else
      Left(InvalidEmail)
  
```

We've introduced an `Either` type which does exactly that, it is Either an Error which we've defined as a sealed trait
and some case objects (think Enum in Java) OR it is a valid `OkHttpResponse`. 

This was another *aha* moment for me. Now the function type tells me exactly what to expect.
Either an Error that is a `InvalidEmail` or `DatabaseDown` or we get HttpResponse.

An either is implemented as two case classes, Left and Right. Left is by convention for errors and Right is for the
happy path.

I've renamed `HttpResponse` to `OkHttpresonse` as of course we'd still send a HttpResponse to the user it would just be
a 4xx or a 5xx.

To do something similar in Java you could use checked exceptions. But the verbosity of handling them at every layer
usually results in the use of runtime exceptions. Runtime exceptions are completely hidden, they circumvent the type
system and make reasoning about code very hard.

But will our new Either type cause the same issues as checked exceptions? Let's see!

## Using Either in function composition

The problem with our `validatePerson` function is it no longer composes with the next function `createPerson`. The
output of `validatePerson` is a `Either[Error, Person]` but the input of `createPerson` is `Person`. 

### Do we need to go and change all our functions? Even the ones that can't fail?

No no no. In functional programming we can `lift` functions. Sounds scary but all it means is
we can take a function with type: `Cat => Dog` and turn it into a function of type `F[Cat] => F[Dog]` where
`F` is something like either. 

There are libraries out there that do this for you but we'll write our own and the final post in this series will use
the libraries I'd recommend  in production code.

```scala
object Lifting {
    def lift[A, B, Error](f: A => B): Either[Error, A] => Either[Error, B] = e => e.map(f)
}
```

Map is a function that transforms the A in either to B if it is a `Right` and just propagates the error if it is a
`Left`.

Pause and think about that. If we map a `Either[DeadAnimal, Cat]` to a `Either[DeadAnimal, Dog]` and the input is a
`Left(DeadAnimal)` then we just get our dead animal back. We essentially do nothing as soon as we get our first `Left`.
However if our cat is still alive and we pass in a `Right(Cat)` then we'll magically turn our cat into a dog and get a
`Right(Dog)`. How you actually convert cats into dogs I'll leave to a scientist.

Enough about cats and dogs. We'll instead use this to take our `createCustomer: Person => Customer` function and turn it into `createCustomer: Either[Error, Person]
=> [Error, Customer]`

And we can do the same for `saveCustomer` and `serialiseCustomer`. Here is the full code now:

```scala
object Errors {
  case class Person(name: String, email: String)
  case class Customer(customerNumber: Int, p: Person)

  sealed trait Error
  case object InvalidEmail extends Error
  case object DatabaseDown extends Error

  type HttpRequest = String
  type OkHttpResponse = String
  type WebRequest = HttpRequest => Either[Error, OkHttpResponse]

  def validatePerson(p: Person): Either[Error, Person] =
    if (p.email.contains("@"))
      Right(p)
    else
      Left(InvalidEmail)


  val serialiseCustomer: Customer => OkHttpResponse = ???
  val deSerialisePerson: HttpRequest => Person = ???


  def createCustomer(p: Person): Customer = {
    // remove the saving logic
    Customer(1, p)
  }

  def saveCustomer(c: Customer): Customer = {
    // do DB stuff, we'll deal with failure in a later post
    c
  }

  import Lifting._

  val registerCustomer: WebRequest =
    deSerialisePerson andThen
      validatePerson andThen
      lift(createCustomer) andThen
      lift(saveCustomer) andThen
      lift(serialiseCustomer)
}

object Lifting {
  def lift[A, B, Error](f: A => B): Either[Error, A] => Either[Error, B] = e => e.map(f)
}
```

Our new function composition lifts all the functions that come after the first function that could fail:

```scala
  val registerCustomer: WebRequest =
    deSerialisePerson andThen
      validatePerson andThen
      lift(createCustomer) andThen
      lift(saveCustomer) andThen
      lift(serialiseCustomer)

```

What's so cool about this is that as soon as the first function fails the rest of the functions are never invoked because of the
way the `map` in our `lift` function works.

### What about the database?

We'll change the type of saveCustomer from `Customer => Customer` to `Customer => Either[Error, Customer]` because we
all know talking to the database can fail.

This again breaks our composition and lift doesn't work on a function that returns an Either so we need a slightly
different lift:

```scala
object Lifting {
  def lift[A, B, Error](f: A => B): Either[Error, A] => Either[Error, B] = e => e.map(f)
  def liftErrorFunction[A, B, Error](f: A => Either[Error, B]): Either[Error, A] => Either[Error, B] = e => e.flatMap(f)
}
```

This new function uses `flatMap` to allow us to convert a function that takes a function of type `A => Either[Error, B]`
and returns a function of type: `Either[Error, A] => Either[Error, B]`

Now our composition works again:

```scala
  val registerCustomer: WebRequest =
    deSerialisePerson andThen
      validatePerson andThen
      lift(createCustomer) andThen
      liftErrorFunction(saveCustomer) andThen
      lift(serialiseCustomer)
```

Now we deal with all the errors and we don't have our code filled with try catch but our types still represent exactly
what is happening under the covers.

All the intent is there without boiler plate!

This is such a common pattern Scala's for comprehension does this for us and
we can delete our lift functions:

```scala
  val registerCustomerMkTwo: WebRequest = in =>  {
    val p = deSerialisePerson(in)
    for {
      person <- validatePerson(p)
      customer = createCustomer(person)
      saved <- saveCustomer(customer)
    } yield serialiseCustomer(saved)
  }
```

Or you can inline the `lift` and `liftErrorFunction` and call `map` and `flatMap` directly.

This style will take some getting used to but in my opinion it is far preferable to using checked exceptions and
shows intent far better than hidden runtime exceptions. For a tiny bit more effort, e.g. the lifting or using a for
comprehension, our types tell us exactly what is going on.


### Extra credit

Either is just one of the types that allow this. There is also:
* Option 
* Try 
* Xor from cats
* \/ from scalaz

Go and find these and re-do the above examples with them.

