# Quick-glance-at-Moq

Moq is a mocking framework built to facilitate the testing of components with dependencies.

As shown earlier, [dealing with dependencies](https://github.com/emgdev/unit-testing-csharp/tree/c1e06f02ecb67288bafa6a2fe26e4d233f910b0e/docs/Dealing-with-dependencies/README.md) could be cumbersome because it requires the creation of test doubles like fakes.

Moq makes the creation of fakes redundant by using dynamically generated types. Its API is built around lambda constructs to ensure tests are strongly typed, easy to read and quick to write. It also provides a string-based API to mock protected methods.

There is an exhaustive Quickstart page available [here](https://github.com/Moq/moq4/wiki/Quickstart) for quick reference.

Note: at the time of writing the latest version of Moq is the 4.14 but a 5.0 is in the writing.

## Example

Let's take as an example the class below. It accepts a dependency of type `IFoo` via its constructor and it uses it in its `Ping` method.

```csharp
public class Service
{
    private readonly IFoo _foo;

    public Service(IFoo foo)
    {
        _foo = foo ?? throw new ArgumentNullException(nameof(foo));
    }

    public void Ping()
    {
        _foo.DoSomething("PING");
    }
}

public interface IFoo
{
    bool DoSomething(string command);
}
```

Let's write a test that uses Moq to verify that the method `Ping` uses the `IFoo.DoSomething` method by passing it a well-known string.

```csharp
[Test]
public void Ping_invokes_DoSomething()
{
    // ARRANGE
    var mock = new Mock<IFoo>();
    mock.Setup(p => p.DoSomething(It.IsAny<string>())).Returns(true);
    var sut = new Service(mock.Object);

    // ACT
    sut.Ping();

    // ASSERT
    mock.Verify(p => p.DoSomething("PING"), Times.Once());
}
```

The unit test in the snippet is quite condensed and few things are happening. Let's look at it line by line.

```csharp
var mock = new Mock<IFoo>();
```

Initially, a new mock is created by specifying the mocked type. In this example, the created object is of type \`Mock´.

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<string>())).Returns(true);
```

Next, the mock is configured by using the method `Setup` to specify incoming parameters and return values.

* The `Setup` construct is used to configure expectations on the invocations of a method of the mocked type. At this time developers can set expectations on the incoming parameters too. In this case, Moq is instructed using the `It.IsAny<string>()` method to accept any string.
* The `Returns` construct is used to specify the return values of the mocked method.

In this case, Moq is instructed so that any invocation \(i.e. any `string`\) of the method `DoSomething` returns `true`. Thanks to Moq's strongly-typed API, it's impossible to configure a method with wrong types.

```csharp
var sut = new Service(mock.Object);
```

Once the mock is configured, the mocked object can be accessed or fed to the system under test's constructor using the `Object` property.

This concludes the _Arrange_ phase of the test.

```csharp
sut.Ping();
```

This line represents the _Act_ phase of the test: the system under test is exerted by invoking its `Ping` method.

```csharp
mock.Verify(p => p.DoSomething("PING"), Times.Once());
```

Finally, we can use the `Verify` method to verify that the expected call had actually happened. In this example, we verify that `DoSomething` was invoked exactly once with the string `"PING"` as parameter.

Once created, a mock keeps track of every invocation on the mocked object and gives developers the possibility to verify that a specific call has occurred.

