Moq has certain behaviors that might appear quite opinionated. Although these behaviors are active by default, they can be customized through special properties exposed by mocks.

## Mock behavior
By default, Moq supports developers by allowing them to create unit tests without forcing them to declare every expected call. This is considered a particularly forgiving behavior because mocking frameworks would usually reject any call that was not configured.

Moq's default behavior enables quicker development of unit tests by  defining a fallback configuration that accepts any parameter and returns empty sequences or the default value.

Sometimes there is a need for a more controlled environment by requiring Moq to attain a more strict behavior. For this reason, when creating a mock, developers can specify the desired behavior and override the default.
```csharp
var mock = new Mock<IService>(MockBehavior.Strict);
```

The `MockBehavior` enumeration has the following values:
- `Strict`: an exception is thrown whenever a method or property is invoked without a matching configuration
- `Loose`: Moq accepts all invocations and attempts to create a valid return value
- `Default`: equivalent to `Loose`.

## Default value
Another behavior of mocks that can be customized is how values are generated when there is no configuration in `Loose` mode.
Moq supports two behaviors: `Empty` and `Mock`.

The first one is the default behavior and was explained earlier: Moq would either return the default value for the specified type or, in case of a sequence of values, an empty sequence.
```csharp
var mock = new Mock<IService> { DefaultValue = DefaultValue.Empty };
```
Alternatively, Moq can return a new mock, assuming the return type is somehow mockable.
```csharp
var mock = new Mock<IService> { DefaultValue = DefaultValue.Mock };
```

## Custom DefaultValueProvider
Rather than relying on the built-in Default Value behaviors presented earlier, developers can customize this aspect by specifying a `DefaultValueProvider`.
```csharp
public class MyCustomDefaultValueProvider : DefaultValueProvider { ... }

var mock = new Mock<IService> { DefaultValueProvider = new MyCustomDefaultValueProvider() };
```
Whilst publicly available, this extension seam should be used sparingly and mostly when integrating with other libraries.