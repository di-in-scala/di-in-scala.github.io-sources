
[Akka](http://akka.io) is a very popular toolkit for building concurrent, "reactive" applications. The main building block used in Akka-based systems are actors. A common question when using Akka is "how do I do dependency injection with actors?"; typical use-case is passing a datasource to an actor.

## Injecting services

The current best-practice for defining actor instantion code, when the actor instance needs some external dependencies, is to provide a method in the companion object returning the `Props` describing how to create a new actor instance (remember, actor instances can be created multiple times, even if it's a "singleton" actor!):

````scala
import akka.actor.{Actor, Props}

class ReactiveTrainDispatch(
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) extends Actor {

   def receive = ...
}

object ReactiveTrainDispatch {
   def props(trainLoader: TrainLoader, trainDispatch: TrainDispatch) = 
      Props(new ReactiveTrainDispatch(trainLoader, trainDispatch))
}
````

When using MacWire, we could already simplify this code a bit by using `wire` inside of `Props`: `Props(wire[ReactiveTrainDispatch])`. However, we still need to repeat all the dependencies in the `props` method signature.

An alternative is to move the props-creating code into a [trait-module](#modules) (assuming that `ReactiveTrainDispatch` lives in the `station` package):

````scala
class ReactiveTrainDispatch(
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) extends Actor {

   def receive = ...
}

trait StationModule {
   lazy val reactiveTrainDispatchProps = 
      Props(wire[ReactiveTrainDispatch]) 

   // as in the previous examples
   lazy val trainDispatch = wire[TrainDispatch]
   lazy val trainStation = wire[TrainStation]

   // dependencies of the module
   def trainShunter: TrainShunter 
   def trainLoader: TrainLoader
}
````

Going further, if the `ActorSystem` is also a value defined in the modules, we can create an actor factory without the need for an intermediate props value (why a factory? As we may want to do an explicit call to create a single or multiple actors):

````scala
class ReactiveTrainDispatch(
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) extends Actor {

   def receive = ...
}

trait StationModule {
   def createReactiveTrainDispatch = 
      actorSystem.actorOf(Props(wire[ReactiveTrainDispatch])) 
   // actor system module dependency

   def actorSystem: ActorSystem

   // as previously
   // ...
}
````

## Injecting other actors 

Apart from services, actors often depend on other actors, holding references to them via `ActorRef`s. The most common way to obtain references to other actors is by sending them via messages. If, however, you want to "inject" an `ActorRef` into your actor at the time of creation, you can do it via a constructor as well.

There's a problem of course with multiple actor references as they all have the same type. With manual DI things are easy - we can just pass the `ActorRef` we need as the proper constructor argument. If, however, we are using MacWire, or if we want the manual approach to be more typesafe, we can use [tagging](#multipleinst).

Again, here we are modelling actor creation as a factory, as actors are typically created explicitly when needed, unlike the "service" object graph, but we could model them as all other dependencies as well:

````scala
class ReactiveTrainDispatch(
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch,
   loadListener: ActorRef @@ LoadListener) extends Actor {

   def receive = ...
}

trait StationModule {
   def createReactiveTrainDispatch(
      loadListener: ActorRef @@ LoadListener) = 
      actorSystem.actorOf(Props(wire[ReactiveTrainDispatch])) 

   // actor system module dependency
   def actorSystem: ActorSystem

   // as previously
   // ...
}

// usage; statically checked ActorRef types!
val loadListener = actorSystem
      .actorOf(Props[LoadListenerActor])
      .taggedWith[LoadListener]

val reactiveTrainDispatch = modules
      .createReactiveTrainDispatch(loadListener)
````
