# Exceptions

One of the most common tasks that were solved with callbacks is throwing an exception when a certain method is invoked with a given set of parameters.

```csharp
var mock = new Mock<IService>();

mock.Setup(p => p.DoSomething())
    .Callback(() => throw new Exception());
```

To streamline the configuration of methods throwing exceptions, Moq has special constructs targeting this scenario: `Throws` and `ThrowsAsync`.

Both methods accept an instance of any Exception type.

```csharp
mock.Setup(p => p.DoSomething())
    .Throws(new Exception("My custom exception"));

mock.Setup(p => p.DoSomethingAsync())
    .ThrowsAsync(new Exception("My custom exception"));
```

Additionally, `Throws` can also instantiate an exception given its type.

```csharp
mock.Setup(p => p.DoSomething())
    .Throws<Exception>();
```

`Thows` and `ThrowsAsync` can also be used in a sequence of calls.

```csharp
mock.SetupSequence(p => p.GetSomeValue())
    .Returns(1)
    .Throws<Exception>();

mock.SetupSequence(p => p.GetSomeValueAsync())
    .ReturnsAsync(1)
    .ThrowsAsync(new Exception());
```

## Throwing exceptions aware of incoming parameters

Unfortunately, `Throws` and `ThrowsAsync` have no overload accepting a delegate to be lazily evaluated.

The lack of such overloads makes it impossible to throw exceptions that are somehow aware of the incoming parameters.

The following statement is not supported.

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<int>()))
    .Throws((int value) => new Exception($"{value} is not valid"));
```

The same behavior can be modeled with callbacks.

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<int>()))
    .Callback((int value) => throw new Exception($"{value} is not valid"));
```

