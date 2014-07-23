
[Dependency Injection (DI)](http://en.wikipedia.org/wiki/Dependency_injection) is a popular pattern implementing inversion of control, which encourages loose coupling between a services’ clients and service implementations.

This guide describes how to do Dependency Injection using the Scala language constructs as much as possible, while remaining practical, with the help of the [MacWire](https://github.com/adamw/macwire) library where necessary. 

Dependency Injection is a *simple* concept, and it can be implemented using relatively few *simple* constructs. We should avoid over-complicating and over-using containers or frameworks, without thoroughly analysing the costs.

<p class="message">
  This guide is <a href="https://github.com/scaladi/scaladi.github.io">available on GitHub</a>, so if you think something is missing, not clear enough, or can be done better, don't hesitate and send a pull request!
</p>

## What is Dependency Injection?

DI is all about decoupling client and service code (the client may happen to be an implementation of another service). Instead of creating hard links to specific service implementations, *references* to services are "injected”. This makes the code easier to understand, more testable and more reusable.

The means of injecting the dependencies vary from approach to approach, but the one we will be using here is passing dependencies through constructor parameters. Other possibilities include setter/field injection, or using a service locator. Hence, the essence of DI can be summarised as *using constructor parameters*.

If you are not yet sold on DI, I recommend reading the [motivation behind Guice](https://github.com/google/guice/wiki/Motivation). It uses Java as the base language, but the ideas are the same and apply universally.

## Other approaches

There are numerous frameworks and approaches implementing DI in various languages and on various platforms. Below are a couple of alternatives to the pure Scala+MacWire approach presented here.

Using frameworks:

* [Subcut](https://github.com/dickwall/subcut): mix of the service locator/dependency injection patterns
* [Scaldi](http://scaldi.org/): similar to Subcut
* [Spring](http://spring.io/): Spring is a very popular Java DI framework (and much more). It also works for Scala.
* [Guice](https://github.com/google/guice): another popular Java DI framework

Using pure Scala:

* [Cake pattern](http://jonasboner.com/2008/10/06/real-world-scala-dependency-injection-di/)
* [Reader monad](http://blog.originate.com/blog/2013/10/21/reader-monad-for-dependency-injection/)

## Running example

To provide code examples throughout the guide, we will need a running example. 

Suppose we are creating a system for a train station. Our goal is to call the method to prepare (load, compose cars) and dispatch the next train. In order to do that, we need to instantiate the following service classes:

````scala
class PointSwitcher()
class TrainCarCoupler()
class TrainShunter(
   pointSwitcher: PointSwitcher, 
   trainCarCoupler: TrainCarCoupler)

class CraneController()
class TrainLoader(
   craneController: CraneController, 
   pointSwitcher: PointSwitcher)

class TrainDispatch()

class TrainStation(
   trainShunter: TrainShunter, 
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) {

   def prepareAndDispatchNextTrain() { ... }
}
````

The dependencies of each class are expressed as constructor parameters. The dependencies form an object graph, which needs to be wired.