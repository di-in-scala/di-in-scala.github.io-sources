
So far all of the dependencies have been declared as `lazy val`s, making them essentially singletons in the scope of a single app usage. Note that these aren’t singletons in the global sense, as we can create multiple copies of the object graph.

However, sometimes we want to create new instances of a dependency for each usage (sometimes called the "dependent scope”). To achieve this, we can simply declare the instance as a `def`, instead of a `lazy val`. If, for example, we needed a new instance of the train dispatcher each time, the code would become:

````scala
object TrainStation extends App {
   lazy val pointSwitcher = wire[PointSwitcher]
   lazy val trainCarCoupler = wire[TrainCarCoupler]
   lazy val trainShunter = wire[TrainShunter]

   lazy val craneController = wire[CraneController]
   lazy val trainLoader = wire[TrainLoader]

   // note the def instead of lazy val
   def trainDispatch = wire[TrainDispatch] 

   // the stations share all services except the train dispatch,
   // for which a new instance is create on each usage
   lazy val trainStationEast = wire[TrainStation]
   lazy val trainStationWest = wire[TrainStation]

   trainStationEast.prepareAndDispatchNextTrain() 
   trainStationWest.prepareAndDispatchNextTrain() 
}  
```` 

Hence, using Scala constructs we can implement two scopes: singleton and dependent.