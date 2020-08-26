Over time developers have agreed on certain best practices. Writing tests to ensure those best practices are followed through can be tedious.

Based on AutoFixture, the classes contained in the package [`AutoFixture.Idioms`](https://www.nuget.org/packages/AutoFixture.Idioms/) help creating unit tests that quickly check if these best practices are properly followed.

The `AutoFixture.Idioms` package contains several assertions. Each assertion encapsulates a unit test that tests a specific expectation on the system under test.

Here is a list of scenarios that can be accellerated by using assertions available in the `AutoFixture.Idioms` package.

## Guard a method against null values

When writing methods or constructors, it is good practice to protect them against unsupported null values. The `GuardClauseAssertion` verifies that the parameters passed to a method or a constructor are properly checked against null values.

Assuming a test class like the following one (for semplicity we will use `string` as dependencies):
```csharp
public class TestClass
{
	private readonly string _firstDependency;
	private readonly string _secondDependency;
	
	public TestClass(string firstDependency, string secondDependency) 
	{
		_firstDependency = firstDependency ?? throw new ArgumentNullException(nameof(firstDependency));
		_secondDependency = secondDependency ?? throw new ArgumentNullException(nameof(secondDependency));
	}

    public void DoSomething(string parameter)
    {
        if (parameter == null) throw new ArgumentNullException(nameof(parameter));

        ...
    } 
}
```
The snippet below will test that every parameter of the constructor is guarded against null values.

```csharp
[Test]
public void Constructor_is_guarded_against_nulls()
{
    var fixture = new Fixture();
    var assertion = fixture.Create<GuardClauseAssertion>();
    assertion.Verify(typeof(TestClass).GetConstructors());
}
```
If any of the nullable parameters of the constructor were not to be guarded, an exception would be thrown and the unit test would fail.

To be noted that without the help of the Idioms package, the developer would expected to write `N+1` unit tests, only for the constructor: one for each incoming parameter and one for the correct initialization of the system under test.

```csharp
[Test]
public void FirstDependency_is_required()
{
    var fixture = new Fixture();
    Assert.That(() => new TestClass(null, fixture.Create<string>()), Throws.ArgumentNullException);
}

[Test]
public void SecondDependency_is_required()
{
    var fixture = new Fixture();
    Assert.That(() => new TestClass(fixture.Create<string>(), null), Throws.ArgumentNullException);
}

[Test]
public void TestClass_can_be_instantiated()
{
    var fixture = new Fixture();
    Assert.That(() => new TestClass(fixture.Create<string>(), fixture.Create<string>()), Throws.Nothing);
}
```

Besides the annoyance of creating multiple repetitive tests, the tests above will need to be fixed every time a new change in the constructor were to occur.

The same assertion can be used to verify the correct implementation of normal methods.

```csharp
[Test]
public void DoSomething_is_guarded_against_nulls()
{
    var fixture = new Fixture();
    var assertion = fixture.Create<GuardClauseAssertion>();
    assertion.Verify(typeof(TestClass).GetMethod(nameof(TestClass.DoSomething)));
}
```

## Constructor initialization

Another good practice is to initialize read-only properties with values passed to the constructor. The `ConstructorInitializedMemberAssertion` can be used to verify that these properties have been initialized with the value passed to the constructor via same-name arguments.

Let´s take this class as test subject.

```csharp
public class TestClass
{
    public TestClass(string parameter, int value)
    {
        Parameter = parameter ?? throw new ArgumentNullException(nameof(parameter));
        Value = value;
    }

    public string Parameter { get; }

    public int Value { get; }
}
```

The `ConstructorInitializedMemberAssertion` can be used to verify that the properties `Parameter` and `Value` contain the same values passed to the constructor.

```csharp
[Test]
public void Properties_are_initialized_by_constructor()
{
    var fixture = new Fixture();
    var assertion = fixture.Create<ConstructorInitializedMemberAssertion>();
    assertion.Verify(typeof(TestClass));
}
```
Commenting any of the two assignments in the `TestClass` constructor will cause the test in the snippet above to fail.

## Value equality

When working with value objects (i.e. objects who are distinguishable by the state of their properties and not by their identity), handling correctly the equality checks is paramount.

In C#, equality checks are customized by overriding the virtual methods `Object.Equals` and `Object.GetHashCode`.

When overriding thesem methods, developers must make sure that the properties of equality are satisfied. These are:
- Reflexive: an object must be equal to itself. `a == a`
- Symmetric: if an object is equal to another, the second is also equal to the first. `(a == b) => (b == a)`
- Transitive: if an object is equal to another, and this is equal to a third, the first is equal to the third. `(a == b, b == c) => (a == c)`

On top of these logic requirements, there are additional requirements:
- A reference type cannot be equal to `null`,
- A class overriding `Object.Equals` must override `Object.GetHashCode` too,
- Applying repetedly `Object.Equals` to the same object must return the same result,
- Applying repetedly `Object.GetHashCode` to the same object must return the same result,
- Testing for equality with `new object()` must return `false`.

Unfortunately, as of today, C# doesn't have a good built-in support for value objects ([see Records coming in C# 9.0](https://devblogs.microsoft.com/dotnet/welcome-to-c-9-0/#records)). This means that a lot of work has to be put to correctly handle value equality.

`AutoFixture.Idioms` contains assertions that can reduce the amount of code needed for the tests.

Specifically, these assertions are available:
- `EqualsNewObjectAssertion` verifies that `Object.Equals` is implemented so that `x.Equals(new object())` is always `false`,
- `EqualsNullAssertion` verifies that `Object.Equals` is implemented so that `x.Equals(null))` is always `false`,
- `EqualsSelfAssertion` verifies that `Object.Equals` is implemented so that `x.Equals(x))` is always `true`,
- `EqualsSuccessiveAssertion` verifies that `Object.Equals` is implemented so that calling `x.Equals(y)` several times returns always the same value,
- `GetHashCodeSuccessiveAssertion` verifies that `Object.GetHashCode` is implemented so that calling `x.GetHashCode()` several times returns always the same value.

As of today, developers are left to write unit tests that prove that the symmetric and transitive properties are respected.