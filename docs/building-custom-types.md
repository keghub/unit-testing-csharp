# Building-custom-types

Creating instances of custom types is where AutoFixture really shines.

By mixing the [default configurations](https://github.com/emgdev/unit-testing-csharp/tree/c1e06f02ecb67288bafa6a2fe26e4d233f910b0e/docs/Default-configurations/README.md) and reflection, AutoFixture is able to generate instances of almost every type, even without an explicit configuration.

```csharp
var fixture = new Fixture();
var person = fixture.Create<Person>();

Assert.That(person, Is.Not.Null);
Assert.That(person.FirstName, Is.Not.Null);
Assert.That(person.LastName, Is.Not.Null);
```

## How custom types are created

When analyzing an unknown type, AutoFixture will use reflection to find the easiest way to create an instance of the type.

* First, it looks for a public constructor, if many constructors are found, the one with fewest parameters are selected. If any parameter is needed, AutoFixture will recursively create instances of the needed types to satisfy the requirements of the selected constructor. If the constructor can't be invoked because its parameters can't be instantiated, the constructors are discarded.
* If no suitable public constructor is found, AutoFixture will look for a static method returning an instance of the type. If multiple methods are found, the one with fewest argument is selected. Like for constructors, AutoFixture will take care of creating instances to be fed as parameters to the selected static method.
* If no static method matching the requirements is found, an exception is thrown.
* Once the object is constructed or obtained from a static factory method, all writeable properties are provided with a value.

## Some examples

Here are some examples on how custom types are built

In the sample below, the parameterless constructor is selected

```csharp
public class TestClass
{
    public TestClass () {}

    public TestClass (int value) { }
}
```

Here, the constructor with fewer arguments is selected

```csharp
public class TestClass
{
    public TestClass(int value) { }

    public TestClass(int value, string anotherValue) { }
}
```

Finally, the static factory method with fewer arguments is used to instantiate the type.

```csharp
public class TestClass
{
    private TestClass () {}

    public static TestClass Create(int value) => new TestClass();

    public static TestClass Create(int value, string anotherValue) => new TestClass();
}
```

