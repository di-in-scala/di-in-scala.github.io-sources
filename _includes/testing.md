
Individual components can be tested by providing mock/stub implementations of some of the dependencies.

Moreover, when using the thin cake pattern, modules can be integration-tested, using the wiring defined in the module.

Of course, we will need to provide some implementation (again, can be a mock/stub), for any dependencies expressed as abstract members. However, it is also possible to override some of the dependencies, to provide alternative implementations for testing. These implementations will be used to wire the graph fragment defined in the module.

For example, to test our shunting module, we could mock the point switcher, which interacts with some external systems, and write an integration test:

````scala
// main code

package shunting {
   trait ShuntingModule {
      lazy val pointSwitcher = wire[PointSwitcher]
      lazy val trainCarCoupler = wire[TrainCarCoupler]
      lazy val trainShunter = wire[TrainShunter] 
   }
} 

// test
class ShuntingModuleItTest extends FlatSpec {
   it should "work" in {
      // given
      val mockPointSwitcher = mock[PointSwitcher]

      // when
      val moduleToTest = new ShuntingModule {
         // the mock implementation will be used to wire the graph
         override lazy val pointSwitcher = mockPointSwitcher
      }
      moduleToTest.trainShunter.shunt()

      // then
      verify(mockPointSwitcher).switch(...)
   }
}
````