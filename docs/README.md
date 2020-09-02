# Unit testing in C\#

## The need for automated testing

Software systems are complex creatures. They are composed of many small components working together to accomplish a greater common goal.

Manually testing all the different use cases with all the possible parameters and making sure all the different expectations are respected would be impossible and quite a tedious job.

Therefore, having a proper suite of automated tests is extremely important for any non-trivial codebase.

It's important to emphasize the difference between the expectations we put on automated test suites. Unlike common knowledge would suggest, automated tests do not test whether the system is working correctly. Automated test suites codify the knowledge we have about the behavior of the system. A failing test doesn't represent a fault in the system. Instead, it represents a fault in our knowledge of the system's actual behavior. This is why development techniques such as Test-Driven Development \(TDD\) and Behavior-Driven Development \(BDD\) have grown in popularity in the recent years.

There are several types of automated tests. The most common ones are unit tests and integration tests.

Unit tests focus on the behavior of individual components by asserting the expected output given a known initial state and a known input.

As the name says, integration tests focus on the integration between multiple components by asserting the expected output of the collaboration of multiple components.

## Objectives of this course

This course focuses on the creation of suites of unit tests aiming at asserting the known behavior of .NET components.

At the end of this course, you should have acquired the knowledge and be acquainted with the tools needed to author and execute unit tests targeting .NET services and components.

In this course we will be using NUnit, Moq and AutoFixture and learn how to glue them together to quickly create powerful test suites that clearly communicate their intent.

After having learned how to use these libraries, we will discuss some advanced scenarios like testing a service consuming a REST API.

Finally, we will look at what code coverage means and how we should look at this metric.

## License

This course is offered under a MIT license that can be consulted in the [GitHub repository](https://github.com/emgdev/unit-testing-csharp) or at [this address](https://raw.githubusercontent.com/emgdev/unit-testing-csharp/master/LICENSE).
