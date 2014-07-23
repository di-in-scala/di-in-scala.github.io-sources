
Often there are cases when we have multiple implementations for some functionality, and we need to choose one depending on configuration. This can be modelled in at least two ways.

Firstly, we can have a single module, which contains conditional logic choosing the right implementation. Suppose we have two options for train shunting, either the traditional one, or using teleportation, and a config flag:

````scala
package shunting {

   trait TrainShunter
 
   class PointSwitcher()
   class TrainCarCoupler()
   class TraditionalTrainShunter(
      pointSwitcher: PointSwitcher,
      trainCarCoupler: TrainCarCoupler) 
      extends TrainShunter

   class TeleportingTrainShunter() extends TrainShunter

   trait ShuntingModule {
      lazy val pointSwitcher = wire[PointSwitcher]
      lazy val trainCarCoupler = wire[TrainCarCoupler]

      lazy val trainShunter = if (config.modern) {
         wire[TeleportingTrainShunter]
      } else {
         wire[TraditionalTrainShunter]
      }  

      def config: Config
   }
}  
````  

Secondly, a module can have multiple implementations. In such a case, we can create an interface-module containing only abstract members, which are implemented in the proper modules. Such an interface-module can also be very useful for expressing dependencies (without relying on a naming convention), and creating very strong links:

````scala
package shunting {

   trait TrainShunter
 
   class PointSwitcher()
   class TrainCarCoupler()
   class TraditionalTrainShunter(
      pointSwitcher: PointSwitcher, 
      trainCarCoupler: TrainCarCoupler) 
      extends TrainShunter

   class TeleportingTrainShunter() extends TrainShunter

   trait ShuntingModule {
      def trainShunter: TrainShunter
   }

   trait TraditionalShuntingModule extends ShuntingModule {
      lazy val pointSwitcher = wire[PointSwitcher]
      lazy val trainCarCoupler = wire[TrainCarCoupler]
      lazy val trainShunter = wire[TraditionalTrainShunter]
   }

   trait ModernShuntingModule extends ShuntingModule {
      lazy val trainShunter = wire[TeleportingTrainShunter]
   } 
}  

// ...

object TrainStation extends App {
   val traditionalModules = new TraditionalShuntingModule
      with LoadingModule
      with StationModule

   val modernModules = new ModernShuntingModule
      with LoadingModule
      with StationModule 

   traditionalModules.trainStation.prepareAndDispatchNextTrain()   
   modernModules.trainStation.prepareAndDispatchNextTrain()   
} 
````   

The downside of this approach is that the module stack must be known at compile time (cannot be chosen dynamically). While it is possible to create 2 or 4 different stacks for a couple of config options, with increasing config options the number of stacks grows exponentially.

The interface-trait-module and implementation-trait-module is in fact part of the approach taken by the Cake Pattern for expressing dependencies. However this results in quite a lot of boilerplate code, so it’s good to use only when needed. 