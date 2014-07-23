
While it would be great to be able to define in a type-safe way the whole object graph for an application upfront, there are cases when it is necessary to access and extend it dynamically. 

First use-case is when integrating with web frameworks. There it is often needed to access a wired instance by-class. Second use-case is dynamically creating instances of classes, which names are only known at run-time, such as plugins. 

Both of these use-cases are realised by the `Wired` class, which can be created given an instance of a module, containing the object graph, using the `wiredInModule` macro. Any `val`s, `lazy val`s and parameter-less `def`s will be available. 

If our train station management application had a plugin system, which could use any of the dependencies in the object graph, we could instantiate the plugins as follows:


````scala
trait TrainStationPlugin {
   def init(): Unit
}

object TrainStation extends App {
   val modules = new ShuntingModule
      with LoadingModule
      with StationModule

   val wired = wiredInModule(modules)

   val plugins = config.pluginClasses.map { pluginClass =>
      wired
         .wireClassInstanceByName(pluginClass)
         .asInstanceOf[TrainStationPlugin]
   }

   plugins.foreach(_.init())

   modules.trainStation.prepareAndDispatchNextTrain()   
}   
```` 

An instance of `Wired` can be also extended with new instances and instance factories, using the `withInstances` and `withInstanceFactory` methods.

For an example of integrating Dependency Injection and MacWire with Play! Framework, see the [Play+MacWire](http://typesafe.com/activator/template/macwire-activator) activator.