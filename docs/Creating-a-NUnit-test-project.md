Before we go deeper into NUnit details, let’s see how to quickly create a unit test project that uses NUnit.

NUnit has good integration with both Microsoft Visual Studio and JetBrains ReSharper but the fastest way to create a test project is by using the `dotnet new` utility that comes with the .NET Core SDK.

Before you can proceed, you need to install the template. You can quickly do so by executing the following command in your preferred shell.
```
dotnet new -i NUnit3.DotNetNew.Template
```
Once you have installed the template, you can simply create a new NUnit-based unit test project by executing
```
dotnet new nunit
```
By default, the project will be created in the current directory and it will be named after its name. Optionally, you can provide the name of the project: this name will also be used to create a directory for the project.
```
dotnet new nunit -n MyFirstTestProject
```
The newly created project will consist of a project file and a sample unit test class.