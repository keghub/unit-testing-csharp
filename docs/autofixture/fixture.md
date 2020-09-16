# Fixture

Despite its complex architecture, AutoFixture exposes a very simple programming interface wrapped by the `Fixture` type.

A `Fixture` uses a graph of `ISpecimenBuilder`s to serve requests to create auto-generated values \(also known as Specimens or [Anonymous Variables](http://blogs.msdn.com/b/ploeh/archive/2008/11/17/anonymous-variables.aspx)\).

Developers can use its parameterless constructor to obtain an instance based on the default configuration. This configuration includes the wiring for the different parts of the framework and configurations for the most common types of the [Base Class Library](https://docs.microsoft.com/en-us/dotnet/standard/framework-libraries#base-class-libraries).

## The `IFixture` interface

`Fixture` implements the `IFixture` interface. The properties and methods of this interface are used to configure the specimens generation and the creation of the specimens via AutoFixture's fluent API.

### Properties of `IFixture`

`IFixture` exposes the following properties

* `Behaviors`: a list of decorators whose execution wraps the execution of each builder
* `Customizations`: a list of customizations that are used to create values before using default builders
* `ResidueCollectors`: a collection of builders to be used as fallback in case no default builder nor customization was able to serve the request
* `OmitAutoProperties`: specifies whether properties should receive a value, default is `false`.
* `RepeatCount`: specifies the amount of items to be added when creating a collection, default is `3`.

### Methods of `IFixture`

* `Build<T>()`: initiates the creation of an instance of the given type ignoring all prior customizations
* `Customize<T>()`: customizes the creation algorithm for all instances of a given type 
* `Customize()`: applies a customization to the current `Fixture`

## The `ISpecimenBuilder` interface

On top of `IFixture`, `Fixture` implements the `ISpecimenBuilder` interface. This interface exposes the `Create` method that is used by more user-friendly versions of this method. More information about the [specimen builders](./extending-autofixture.md#specimen-builders) and the [`Create` method](./create-and-build.md) will follow.

## Extension methods

On top of the methods above, several extension methods are defined on the `IFixture` type. These extension methods are all built on the basic operations exposed by `IFixture`. The most important ones will be presented in depth later on.

[Here you can have a glance at the internal architecture of AutoFixture](https://github.com/AutoFixture/AutoFixture/wiki/Internal-Architecture)

