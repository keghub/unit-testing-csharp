There are several ways to execute the tests of a suite.

## .NET Core SDK

The .NET Core SDK includes a test runner utility able to execute all compatible test suites: `dotnet test`. More information on how to create test projects using the `dotnet` CLI will [follow](Creating-a-NUnit-test-project).

To execute all the tests in a suite project, simply execute the following command in the preferred shell
```
dotnet test
```
This command will take care of building the source code, if it was needed.

The test utility supports several flags: a complete list can be accessed by executing `dotnet test -h` in the console.

### Automatic execution

The .NET Core SDK includes a file system watcher utility that automatically executes a command when a file in a watched folder is modified. This utility can be used to automatically execute all tests when a file is modified.

To use the file system watcher, simply execute the following command in your preferred shell while being in the folder of your test suite project
```
dotnet watch test
```

## Visual Studio Code

Visual Studio Code is a lean code editor that relies on add-ons (also known as extensions) to support the workloads of the user.

When working with C# solutions, developers should install the [official extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

This extension extends the editor providing lightweight development tools for .NET Core, including a basic support for unit testing. Specifically, this extension gives the developer the possibility to run or debug specific tests or set of tests.

Visual Studio Code also embeds a terminal. This terminal can be used to run tests from the command line like explained earlier.

Finally, the [.NET Core Test Explorer](https://marketplace.visualstudio.com/items?itemName=formulahendry.dotnet-test-explorer) extension adds to Visual Studio Code a panel listing all the unit tests available in the folder.

## Visual Studio 2019

Unlike Visual Studio Code, Visual Studio has a built-in support for C# and unit tests written in this language.

Among the built-in amenities offered by Visual Studio there is the [Test Explorer](https://docs.microsoft.com/en-us/visualstudio/test/run-unit-tests-with-test-explorer?view=vs-2019): a panel that displays all the tests available in the solution. From there, developers can run tests or group of tests and see their outcome.

Another functionality offered by Visual Studio 2019 is [Live Unit Testing](https://docs.microsoft.com/en-us/visualstudio/test/live-unit-testing?view=vs-2019): this functionality executes your unit tests automatically and in real time as you make code changes.