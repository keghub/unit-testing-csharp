# Implicit mocks

Sometimes, Moq's default behaviors are just enough to proceed with the unit test. In this case, Moq offers utility methods to quickly generate a mocked object using the static factory `Mock.Of<T>`.

Implicit mocks are the most useful when dealing with interfaces that don't need any customization nor verification. Unlike explicitly creating a mock using the `Mock<T>` class constructor, `Mock.Of<T>` returns directly the mocked type.

```csharp
var logger = Mock.Of<ILogger>();
```

The line above is fully equivalent to:

```csharp
var mock = new Mock<ILogger>();
var logger = mock.Object;
```

## Accessing the underlying mock

Sometimes, it can be useful to access the mock a mocked object. This can be achieved with the `Mock.Get` utility.

```csharp
var mock = Mock.Get(logger);
```

`Mock.Get` can also be used when accessing the underlying mock of a hierarchy of properties.

```csharp
var mock = new Mock<HttpContextBase>();
var context = mock.Object;
var mockRequest = Mock.Get(context.Request);
```

