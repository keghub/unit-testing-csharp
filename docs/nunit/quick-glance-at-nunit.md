# Quick glance at NUnit

NUnit is an open-source unit testing framework for the .NET platform. It supports multiple runtimes like .NET Framework, .NET Core, Mono and Xamarin.

As the name suggests, it’s part of the xUnit family of testing frameworks and it shares with them the basic concepts and architecture.

Here are some of the basic concepts that can be found in NUnit

* A **test fixture** is a class marked with the TestFixture attribute and contains one or more tests. Test fixtures can be configured with setup and teardown steps that must be performed for each test.
* A **test** is a method marked with the Test attribute
* A **test runner** is an application responsible for executing all the tests and reporting the test results

NUnit has a strong support for data driven tests: the same unit test can be run with multiple input data. We will see how we can use this to create leaner and more effective unit tests.

Finally, to reduce the amount of code repeated for each unit test, NUnit offers different utilities to set-up and tear-down the execution context.

Note: at the time of writing, the latest version of NUnit is the 3.12.

