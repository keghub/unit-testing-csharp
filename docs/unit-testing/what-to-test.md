# What to test

Authoring and maintaining unit tests require effort from the development team. Therefore, it’s particularly important to understand where to focus this effort.

You want to focus your testing efforts on the code you own and maintain: "test only what you own".

Ideally, you want the whole codebase to be covered but this might not be possible, especially for existing codebases with little-to-no coverage. In cases like this, it’s best to understand which parts of the system are critical for your business and focus your testing effort there. Another challenge of maintaining a unit test suite is caused by high entanglement between the business logic and the intrinsic of the used frameworks.

A trivial example can be when too much business logic is placed in UI constructs like MVC controllers or MVVM view models. Especially in the case of older frameworks like ASP.NET, these constructs rely heavily on many components that make testing difficult or meaningless, due to the need of watering down your test spectrum.

The best way to deal with older frameworks is to avoid dealing with them. You can achieve that by moving all the business logic in UI-agnostic services whose role is simply to process the inputs and return the outputs. Similarly, the role of those UI constructs like the ASP.NET controllers will be limited to validating the inputs, coordinating business logic services, and assembling the outputs in a format specific for the underlying client technology.

## Testing external libraries

A valid exception to the rule of “testing only what you own” is when you create a set of tests to assert the expected behavior of a new library you intend to introduce into your codebase: in this way, you can quickly capture alterations of the behavior of the library introduced by a new version. If you were wondering, these are not integration tests since we’re not testing the integration of our code with the external library.

## Using tests to solve and document bugs

Unit tests can be used while solving bugs in a source base.

In this case, the workflow is similar to test-driven development techniques.

The developer assigned to solve an issue will have to go through the following steps:

* Create a unit test that reproduces the expected behavior of the component. This unit test will fail because the system is not behaving as expected.
* Solve the bug in the system under test so that the unit test passes.

The advantages of this approach are:

* The codebase retains an history of the solved bugs in the form of unit tests
* Every solved bug helps expanding the knowledge of the behavior of the component
* The new unit tests protect against eventual regressions

