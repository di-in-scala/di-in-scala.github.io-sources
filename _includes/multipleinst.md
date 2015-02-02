
In some cases we have a couple of objects of the same type that we want to use as dependencies. For example, assume that our train station now needs two train loaders, one for regular freight, one for liquid freight:

````scala
class TrainStation(
   trainShunter: TrainShunter, 
   regularTrainLoader: TrainLoader,  
   liquidTrainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) { ... }
````

In our wiring code we then need to create the two instances:

````scala
lazy val regularTrainLoader = new TrainLoader(...)
lazy val liquidTrainLoader = new TrainLoader(...)
lazy val trainStation = new TrainStation(
   trainShunter,
   regularTrainLoader,
   liquidTrainLoader)
````

This is a perfectly good solution if we are using manual dependency injection - everything works as expected. One downside is that it's not entirely type-safe: if we mix-up the regular and liquid train loaders when passing them as arguments to the `TrainStation` constructor, everything will still compile just fine.

However, if we try to use MacWire (or implicits), compilation will fail: MacWire has no chances of knowing, which instance should be used where. We need to somehow differentiate the instances so that the compiler will be able to tell which goes where.

## Using different types

The first solution is to give the two dependencies distinct types, however of course that is not always possible. In our case, these might be for example traits, or subclasses:

````scala
class TrainStation(
   trainShunter: TrainShunter, 
   regularTrainLoader: TrainLoader with Regular,  
   liquidTrainLoader: TrainLoader with Liquid, 
   trainDispatch: TrainDispatch) { ... }

lazy val regularTrainLoader = new TrainLoader(...) with Regular
lazy val liquidTrainLoader = new TrainLoader(...) with Liquid
lazy val trainStation = wire[TrainStation]
````

This is also a type-safe solution, we are now **not** able to mix up the two dependencies when passing them as arguments to `TrainStation`.

## Using qualifiers/tags

Another solution is to use tagging from MacWire, which is inspired by the work of [Miles Sabin](https://gist.github.com/milessabin/89c9b47a91017973a35f) and what is present in [Scalaz](https://github.com/scalaz/scalaz). 

A tag is a Scala trait, usually an empty one. An instance of type `X` tagged with tag `T` has type `X @@ T`, or `Tagged[X, T]`, depending which syntax you prefer. You can add tags to instances using the `x.taggedWith[T]` method available on any type. All these declarations can be brough into scope by importing `com.softwaremill.macwire._`.

The tags follow proper subtyping rules: `X <: X @@ T`, so you can use a tagged instance if an untagged one is required. On the other hand, `X @@ T1` is not a subtype of `X @@ T2` if `T1` and `T2` are distinct, so you are safe from using the wrong instance with the wrong tag. 

Tags incur zero runtime overhead, they are a purely compile-time construct.

Our train station dependencies and wiring can then be expressed in the following way:

````scala
trait Regular
trait Liquid

class TrainStation(
   trainShunter: TrainShunter, 
   regularTrainLoader: TrainLoader @@ Regular,  
   liquidTrainLoader: TrainLoader @@ Liquid, 
   trainDispatch: TrainDispatch) { ... }

lazy val regularTrainLoader = wire[TrainLoader].taggedWith[Regular]
lazy val liquidTrainLoader = wire[TrainLoader].taggedWith[Liquid]
lazy val trainStation = wire[TrainStation]
````

Note that you can use any form of "tagging" instances (e.g. the one from Scalaz), MacWire does not require to use the tags from its distribution. The only thing MacWire does during wiring is regular Scala subtype checks.