# Qualities of a good unit test suite

## Fast to run

Depending on the size of the library under test, a comprehensive suite of unit tests might include hundreds or even thousands of tests.

Since developers should feel comfortable running the full suite as often as they can, it's important that the execution time of each unit test is in the range of milliseconds.

Such a fast execution is possible by excluding all dependencies, especially those performing I/O operations \(file and network access mostly\).

If not avoidable, tests that take a lot of time should be executed during the phases of the CI/CD pipeline, at least.

## Coherently structured

Another challenge correlated to the size of the library under test is the ability to quickly find tests relevant to each component of the library.

It is usually suggested having the test library reflecting the file system structure of the target library and name the classes/files containing the tests so that the connection to the tested component is clear and unequivocal.

## Easy to run

Tests are meant to be executed. Making sure that developers have the tools to quickly execute the tests is capital.

Ideally, unit tests should be run every time the developer modifies a file. Each IDE has different ways of supporting this. For example, Visual Studio supports a feature called Live Unit Tests that automatically execute tests as the source code is being written.

Finally, tests’ execution should be part of the CI/CD pipeline.

## Thorough

A test suite should be thorough by not focusing only on happy paths \(the default scenario featuring no exceptional or error conditions\).

Unit tests should focus on the boundaries of the problem space by asserting the behavior of the components when exposed to illegal arguments \(i.e. nulls, out-of-range, unexpected combinations of parameters\).

