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
_WIP_

## Visual Studio 2019
_WIP_