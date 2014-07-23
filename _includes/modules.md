
## Thin cake pattern

At some point, creating the whole object graph at "the end of the world” will become unpractical, and the code large and hard to read. We should then somehow divide it to smaller pieces. Luckily, Scala’s `trait`s fit perfectly for that task; they can be used to split the object graph creation code.

In each trait, which for purpose of this task is also called a "module”, part of the object graph is created. Everything is later re-combined by putting all the necessary traits together.

There may be various rules on how to divide code into modules. A good place to start is to consider creating a pre-wired module per-package. Each package should contain a group of classes sharing or implementing some specific functionality. Most probably these classes cooperate in some way, and hence can be wired.

The additional benefit of shipping a package not only with the code, but also with a wired object graph fragment, is that it is more clear how the code should be used. There are no requirements on actually using the wired module, so if needed, wiring can be done in a different way.

However, such modules can’t usually exist stand-alone: very often they will depend on some classes from other modules. There are two ways of expressing dependencies.

## Expressing dependencies via abstract members

As each module is a trait, it is possible to leave some dependencies undefined, as abstract members. Such abstract members can be used when wiring (either manually or using the `wire` macro), but the specific implementation doesn’t have to be given.

When all the modules are combined in the end application, the compiler will verify that all such dependencies defined as abstract members are defined.

Note that we can declare all abstract members as `def`s, as they can be later implemented as `val`s, `lazy val`s, or left as `def`s. Using a `def` keeps all options possible.

The wiring for our example code can be divided as follows; the classes are now grouped into packages:

````scala
package shunting {
   class PointSwitcher()
   class TrainCarCoupler()
   class TrainShunter(
      pointSwitcher: PointSwitcher, 
      trainCarCoupler: TrainCarCoupler)
} 

package loading {
   class CraneController()
   class TrainLoader(
      craneController: CraneController, 
      pointSwitcher: PointSwitcher)
}

package station {
   class TrainDispatch()

   class TrainStation(
      trainShunter: TrainShunter, 
      trainLoader: TrainLoader, 
      trainDispatch: TrainDispatch) {

      def prepareAndDispatchNextTrain() { ... }
   }
} 
````

Each package has a corresponding trait-module. Note that the dependency between the `shunting` and `loading` packages is expressed using an abstract member:


````scala
package shunting {
   trait ShuntingModule {
      lazy val pointSwitcher = wire[PointSwitcher]
      lazy val trainCarCoupler = wire[TrainCarCoupler]
      lazy val trainShunter = wire[TrainShunter] 
   }
}

package loading {
   trait LoadingModule {
      lazy val craneController = wire[CraneController]
      lazy val trainLoader = wire[TrainLoader] 
 
      // dependency of the module
      def pointSwitcher: PointSwitcher
   }
}

package station {
   trait StationModule {
      lazy val trainDispatch = wire[TrainDispatch]

      lazy val trainStation = wire[TrainStation]

      // dependencies of the module
      def trainShunter: TrainShunter 
      def trainLoader: TrainLoader
   }
}

object TrainStation extends App {
   val modules = new ShuntingModule
      with LoadingModule
      with StationModule

   modules.trainStation.prepareAndDispatchNextTrain()   
}  
```` 

To implement dependencies this way a consistent naming convention is needed, as the abstract member is reconciled with the implementation by-name. Naming the values same as the classes, but with the initial letter lowercase is a good example of such a convention.

This approach is in some parts similar to the Cake Pattern, hence the name: **Thin Cake Pattern**.

## Expressing dependencies via self-types

Another way of expressing dependencies is by using self-types or extending other trait-modules. This way creates a much stronger connection between the two modules, instead of the looser coupled abstract member approach, however in some situations is desirable (e.g. when having a module-interface with multiple implementations, see below).

For example, we could express the dependency between the shunting and loading modules and the station module by extending trait-module, instead of using the abstract members:

````scala
package shunting {

   trait ShuntingModule {

      lazy val pointSwitcher = wire[PointSwitcher]
      lazy val trainCarCoupler = wire[TrainCarCoupler]
      lazy val trainShunter = wire[TrainShunter] 
   }
}

package loading {
   trait LoadingModule {

      lazy val craneController = wire[CraneController]
      lazy val trainLoader = wire[TrainLoader] 
 
      // dependency expressed using an abstract member
      def pointSwitcher: PointSwitcher
   }
}

package station {
   // dependencies expressed using extends
   trait StationModule extends ShuntingModule with LoadingModule {
      lazy val trainDispatch = wire[TrainDispatch]

      lazy val trainStation = wire[TrainStation]

   }
}

object TrainStation extends App {
   val modules = new ShuntingModule
      with LoadingModule
      with StationModule

   modules.trainStation.prepareAndDispatchNextTrain()   
}   
```` 

A very similar effect would be achieved by using a self-type. 

This approach can also be useful to create bigger modules out of multiple smaller ones, without the need to re-express the dependencies of the smaller modules. Simply define a bigger-module-trait extending a number of smaller-module-traits.