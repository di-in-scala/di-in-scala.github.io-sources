
An approach that is often dismissed too easily is doing dependency injection by hand. While this requires a bit more typing, when doing things manually you are free from any constraints that a framework may impose, and hence a lot of flexibility is gained.

When using DI, we have to somehow create (wire) the object graph, that is to create instances of the given classes with the appropriate dependencies. When using framework, this task is delegated to a container. But nothing keeps us from simple writing the code!

The object graph should be created as late as possible. The "end of the world” is the main class of our application. If you have used DI containers before, when using manual DI with or without MacWire, you will soon re-discover the benefits of having a main method in your application.

To be more specific, here’s a manual-DI version for our example:

````scala
object TrainStation extends App {
   val pointSwitcher = new PointSwitcher()
   val trainCarCoupler = new TrainCarCoupler()
   val trainShunter = new TrainShunter(
      pointSwitcher, trainCarCoupler)

   val craneController = new CraneController()
   val trainLoader = new TrainLoader(
      craneController, pointSwitcher) 

   val trainDispatch = new TrainDispatch()

   val trainStation = new TrainStation(
     trainShunter, trainLoader, trainDispatch)

   trainStation.prepareAndDispatchNextTrain()
}
````

## Advantages of Manual DI

The first advantage of the approach presented above is type-safety: the dependency resolution is done at compile-time, so you can be sure that all dependencies are met.

No run-time reflection is needed, which has a slight startup-time benefit (no need to scan the classpath), but also removes a lot of "magic” from the code. We are only using plain Scala, and constructor parameters. It is possible to navigate to the place where the instance is created. The process of creating the object graph is clear. The application is also simple to use, and can be easily packaged e.g. as a fat-jar. No containers to start, no frameworks to fight.

If creating an instance of some object is complex, or choosing an implementation depends on some configuration, thanks to the flexibility of manual DI, we can easily run arbitrary code which should compute the dependency to use.

## `val` vs. `lazy val` 

Defining dependencies using `val`s has one drawback: if a dependency is used, before being initialised, it will be `null` when referenced. That is because `val`s are calculated from top to bottom.

This can be solved by using `lazy val`s, which are calculated on-demand, and the right initialisation order will be calculated automatically.

Hence our manual-DI example becomes:

````scala
object TrainStation extends App {
   lazy val pointSwitcher = new PointSwitcher()
   lazy val trainCarCoupler = new TrainCarCoupler()
   lazy val trainShunter = new TrainShunter(
      pointSwitcher, trainCarCoupler)

   lazy val craneController = new CraneController()
   lazy val trainLoader = new TrainLoader(
      craneController, pointSwitcher) 

   lazy val trainDispatch = new TrainDispatch() 

   lazy val trainStation = new TrainStation(
      trainShunter, trainLoader, trainDispatch) 

   trainStation.prepareAndDispatchNextTrain() 
}
```` 