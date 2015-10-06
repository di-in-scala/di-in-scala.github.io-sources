
## Factories as functions

In the simplest form, a factory is a function: a parametrized way to create an object. You can also think of it as a partially wired dependency.

Let's say the `TrainLoader` additionally requires a `CarLoader` dependency parametrized by `CarType` (the type of the train car to load: coal, refrigerated, chemical etc.). As a convenience, we can create a type alias for the function, but that is entirerly optional. The definition of the `TrainLoader` class now becomes:

````scala
type CarLoaderFactory = CarType => CarLoader

class TrainLoader(
   craneController: CraneController, 
   pointSwitcher: PointSwitcher,
   carLoaderFactory: CarLoaderFactory)
````

We can now wire `TrainLoader` as usual (either manually or using MacWire), however of course we need a dependency of type `CarLoaderFactory` defined somewhere; extending the `LoadingModule` we get:

````scala
trait LoadingModule {
   lazy val craneController = wire[CraneController]
   lazy val trainLoader = wire[TrainLoader] 

   lazy val carLoaderFactory = (ct: CarType) => wire[CarLoader]
   // the above wire will expand to: new CarLoader(ct). Can also 
   // be any other logic to instantiate a CarLoader

   // dependency of the module
   def pointSwitcher: PointSwitcher
}
````

## Factories in trait-modules

You can also create factories by defining methods directly in the trait-modules. The method parameters will be also used for wiring when using `wire[]`.

Let's say the `TrainStation` requires a name:

````scala
class TrainStation(
   name: Name,
   trainShunter: TrainShunter, 
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) {

   def prepareAndDispatchNextTrain() { ... }
}
````

We can expose a parametrized train station in the module by defining a method with a name parameter:

````scala
trait StationModule {
      def trainStation(name: Name) = wire[TrainStation]

      lazy val trainDispatch = wire[TrainDispatch]

      // dependencies of the module
      def trainShunter: TrainShunter 
      def trainLoader: TrainLoader
}
````

When retrieving an instance of a train station from the combined modules, we now have to provide a name. The method parameters, together with other dependencies looked up in the usual way, will be used to create a `TrainStation` instance.

## Wiring using factory methods

Using MacWire, it is also possible to wire objects with a factory method instead of a constructor. For example, suppose that instances of `TrainLoader` needs to be created using a special method which passes some numeric parameters to the crane; in such situations, we can use `wireWith`:

````scala
package loading {
   class TrainLoader(
      craneController: CraneController, 
      pointSwitcher: PointSwitcher,
      xAxisCoefficient: Double,
      yAxisCoefficient: Double)

   object TrainLoader {
      def createDefault(
         craneController: CraneController, 
         pointSwitcher: PointSwitcher) =
         new TrainLoader(craneController, pointSwitcher,
            10.0, 12.5) 
   }  

   trait LoadingModule {
      lazy val craneController = wire[CraneController]

      lazy val trainLoader = wireWith(TrainLoader.createDefault) 

      // dependency of the module
      def pointSwitcher: PointSwitcher
   } 
}
}
````
