# Qualities of a good unit test

## Focused

A unit test should be focused on testing a single component, often referred to as “system under test” or SUT.

At the same time, each unit test should be focused on a single method of the system under test \(unless the interaction between several methods is being tested, which is rare\).

The best way to ensure that a unit test is “focused enough” is by following the “one reason to fail” principle: when a unit test fails, it should be easy to understand which part of the system under test caused the failure.

## Easy to read

Unit tests should be easy to read and clear in their intent so that the developers can easily understand what is being tested under which conditions. To facilitate this, it’s best to structure the unit test according to the AAA pattern.

A unit test should be composed of three parts:

* Arrange
* Act
* Assert

The first section, **Arrange**, is where the test is being set up.

Here are some typical actions in the Arrange sections:

* Prepare the input to be fed to the system under test
* Instantiate the system under test

The second section, **Act**, consists of a single call on the system under test.

The third section, **Assert**, consists of a coherent set of assertions on the output or the state of the system under test after being acted upon. Multiple assertions in a unit test are possible as long as they test the same feature/behavior. If you think that you are violating the “one reason to fail” principle, split the test into two pieces.

## Named consistently

Naming plays an important role.

Unit tests should be named so that relevant information is available at a quick glance. Depending on which programming language you are using, you might be forced to follow some naming rules when creating unit tests.

In C\#, unit tests must follow the method naming syntax rules. A way to convey all the relevant information following the C\# method naming rules, is the following: `<MethodName>_should_<expectation>_when_<condition>`

Some examples

* `Constructor_should_throw_when_parameters_are_null`
* `Echo_should_return_incoming_message`
* `Echo_should_throw_when_incoming_message_is_null`

## Fast

As mentioned earlier, unit tests need to be fast.

The best way to achieve a lightning fast execution is to shed all the dependencies and focus the testing effort to only the logic. Components developed following the SOLID principles are generally easier to test without dependencies on time consuming operations \(network and I/O mostly\).

## Isolated

Unit tests should not rely on each other. Their order of execution should not affect the success or failure.

Unit tests tend to affect each other when they use shared resources. Developers should ensure that the state of the system is cleaned after the execution.

Unit test frameworks support setup and teardown procedures to help development of isolated tests.

In C\#, static object instances must be explicitly handled to ensure each test has a clean execution environment.

## Repeatable

Tests must be able to be run repeatedly without intervention and should produce the same results each time, every time.

Unit tests should rely as little as possible on a specific environment and should not depend on external services or resources that might not always be available.

Tests that sometimes fail are often referred to as “flaky tests”. In cases like this, the developers should spend time to understand what causes the flakiness and possibly split the unit test into two or more.

Also, unit tests should not rely on uncontrollable parameters. Randomly generated parameters are ok under the assumption that all the generated values have the same semantic.

For example, while testing a method that validates a password by its length enforcing a requirement of at least 16 characters, developers should have a test where randomly generated passwords always have more than 16 characters and with random passwords shorter than 16 characters. A test without these constraints would fail or succeed depending on the length of the generated password.

## Self-validating

The output of a test should be unambiguous: either pass or fail. When all tests are green, developers should feel confident that the tested components are behaving as expected. The "one reason to fail" does not only set an upper boundary, but also a lower boundary. A test that can never fail is useless and should be removed.

## Meaningful and unique

In all their goodness, unit tests still represent additional code to maintain.

For this reason, it's important to aim to have all and only meaningful unit tests. Also, each unit test should be unique.

A meaningful unit test is a test that

* exerts the system under test: a unit test that doesn't touch the system under test is irrelevant.
* can fail if certain conditions are not met: a unit test that will never fail is irrelevant.
* has strong expectations on the output

A unit test is unique when there are no other tests that

* have same assumption on the initial state
* exert the system under test with similar inputs
* have similar expectations on the output

Assuming a service that returns the capitalized version of a string, here are some examples of meaningless tests

* a test asserting that the length of the output string is the same of the length of the input string
* a test asserting that the object is not null after properly constructing it
* a test that automatically passes

