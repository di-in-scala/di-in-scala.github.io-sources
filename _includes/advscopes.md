
Especially in web applications, it is useful to have scopes other than singleton and dependent - e.g. scopes tied to the duration of a request, or scopes tied to a user sessions. 

Like interceptors, a lot of the DI containers/frameworks contain support for different scopes. They may seem "magical”, however they are in fact pretty simple.

MacWire contains a general skeleton for defining scopes, similar to interceptors. The `Scope` trait defines two methods: `apply`, which when applied to an instance should create a scoped value, and `get`, to get the current underlying value of the scope. The scope’s life cycle is entirely managed by the implementor.

Similarly to interceptors, usage of scopes is declarative, by using an abstract `Scope` value, and the definition can be provided as late as in the main application entry point.

For example, in a Java servlet-based web project, to make the train dispatch session-scoped (new dispatch for each session), we would first need to declare the usage of the session scope:

````scala
package station {
   trait StationModule extends ShuntingModule with LoadingModule {
      lazy val trainDispatch = session(wire[TrainDispatch])
      lazy val trainStation = wire[TrainStation]

      def session: Scope

   }
}
````  

When starting the application, we need to provide the implementation of the scope. Two implementations are shipped by default, a `NoOpScope` (useful for testing), and a `ThreadLocalScope`, which holds the "current” scoped value in a thread-local variable (hence this implementation is only useful for synchronous web frameworks); the thread-local scope needs to be associated with a storage before each request:

````scala
object TrainStation extends App {
   val modules = new ShuntingModule
      with LoadingModule
      with StationModule {

      lazy val session = new ThreadLocalScope
   }

   // implement a filter which attaches the session to the scope
   // use the filter in the server

   modules.trainStation.prepareAndDispatchNextTrain()   
}   
````

For an example session scope implementation, see the [MacWire](https://github.com/adamw/macwire) site.