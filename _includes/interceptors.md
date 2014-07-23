
Interceptors are very useful for implementing cross-cutting concerns, and are a part of almost every DI framework/container. While there’s no direct support for interceptors in Scala, with a thin library layer (provided by MacWire), it is easy to write and use interceptors.

Using interceptors is a two-step process. First, we have to declare *what* should be intercepted. Ideally, this shouldn’t involve the implementation of the interceptor in any way. Secondly, we have to define what the interceptor does - the behaviour.

To implement the first part, we will define an abstract interceptor, and apply it to selected values. Let’s say that we want to audit all point switches and car couplings events to some external system. To do that, we need to intercept all method calls on the `PointSwitcher` and `TrainCarCoupler` services:

````scala
package shunting {
   trait ShuntingModule {
      lazy val pointSwitcher = logEvents(wire[PointSwitcher])
      lazy val trainCarCoupler = logEvents(wire[TrainCarCoupler])
      lazy val trainShunter = wire[TrainShunter] 

      def logEvents: Interceptor
   }
}
````

We have *declared* that we want to apply the `logEvents` interceptor to the `pointSwitcher` and `trainCarCoupler` services. Note that so far the implementation hasn’t been mentioned in any way. We are only using the abstract `Interceptor` trait, which has an `apply` method, returning an instance of the same type, as passed to it through the parameter.

At some point we of course have to specify the implementation. We can do this as late as possible, the last point being the main entry point to the application: 

````scala
object TrainStation extends App {
   val modules = new ShuntingModule
      with LoadingModule
      with StationModule {

      lazy val logEvents = ProxyingInterceptor { ctx =>
         println("Calling method: " + ctx.method.getName())
         ctx.proceed()
      }
   }

   modules.trainStation.prepareAndDispatchNextTrain()   
}   
````  

Here we have specified that we want to create a proxying interceptor (which will create a Java proxy), with the given behaviour on method invocation. Note that we could use any of the services defined in the modules, when handling the proxied call.

For testing, it may be useful to skip interceptors. This can also easily be done by providing a no-op interceptor implementation:

````scala
class ShuntingModuleItTest extends FlatSpec {
   it should "work" in {
      // given
      val moduleToTest = new ShuntingModule {
         lazy val logEvents = NoOpInterceptor
      }

      // ...
   }
} 
```` 