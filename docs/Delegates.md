Moq can be used to create fake delegates. Mocks faking delegates can be configured like normal methods to [return a value](Results), [execute a callback](Callbacks) or [throw an exception](Exceptions).

The snippets below are based on the following type
```csharp
public delegate int ParseString(string value);
```

Moq can fake explicit delegates by simply passing their type to the constructor.
```csharp
var mock = new Mock<ParseString>();
```

Alternatively, Moq supports delegates based on `Func` and `Action`.
```csharp
var mock = new Mock<Func<string, int>>();
```

It is possible to configure the delegates as if they were normal methods.

```csharp
mock.Setup(p => p(It.IsAny<string>()))
    .Returns(42);
```

Finally, Moq can generate [implicit mocks](Implicit-mocks) for delegates too.
```csharp
var parser = Mock.Of<ParseString>();

var func = Mock.Of<Func<string, int>>();
```