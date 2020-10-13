# Generic methods

Generic methods require type parameters to be specified to be invoked. When dealing with dependencies exposing generic methods, developers can use Moq to configure these methods accordingly by placing constraints on the incoming type parameters.

The snippets in this section will be based on the interface below:

```csharp
public interface IService
{
    void DoSomething<T>(T argument);
}
```

## Expecting any type

Like for configuring methods by accepting any parameters, methods can be configured to accept any type. This can be done with the constant `It.IsAnyType`.

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<It.IsAnyType>()))
    .Callback((object value) => TestContext.Progress.Writeline($"DoSomething: {value}"));
```

## Setting expectations on incoming type parameter

Sometimes, the configuration needs to be more specific than what is allowed from the compiler. In this case, developers can use `It.IsSubType<T>` to constrain the incoming type to the same type or any of its subtypes. This allows different setups for different incoming types

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<It.IsSubtype<IList<string>>>()))
    .Callback((IList<string> items) => TestContext.Progress.Writeline($"Received list of {items.Count} strings"));

mock.Setup(p => p.DoSomething(It.IsAny<It.IsSubtype<IList<int>>>()))
    .Throws<ArgumentException>();
```

