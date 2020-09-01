# Anatomy of a test fixture

We already saw that a test fixture is a class decorated with the `TestFixture` attribute and tests are public methods decorated with the `Test` attribute.

Even by most conservative estimations, test fixture classes tend to be multiple times bigger than the tested component. For this reason, it's better to structure the test fixtures to increase readibility.

Following the common C\# coding style, a test fixture should be structured as follow

* private fields
* test fixture constructor
* test fixture set up methods
* tests targeting constructors of the system under test
* tests targeting methods of the system under tests, grouped by method
* tests targeting properties of the system under tests, grouped by properties
* test fixture tear-down methods

Additionally, local helper methods should be placed next to the method using it. If a helper method is shared by multiple tests, it should be placed at the end of the class.

