When defining argument expectations, developers can use It.Is<T> to apply custom predicates to incoming parameters. 
```csharp
mock.Verify(p => p.DoSomething(It.Is<string>(s => s.Length > 100)));
```
Over time, developers might want to collect these custom expressions into a library. Unfortunately, the matching expressions can't be extracted as they are because `It.Is<T>` accepts an `Expression<Func<T, bool>>` instead of a simple `Func<T, bool>` delegate.

To support this use case, Moq gives developers the possibility to create **custom matchers**. A custom matcher is an expression that wraps a delegate so that it can be used when defining an argument expectation.
```csharp
public static class ParameterExpectations
{
    [Matcher]
    public static string StringLongerThan(int size) => Moq.Match.Create<string>(s => s.Length > size);
}

mock.Verify(p => p.DoSomething(ParameterExpectations.StringLongerThan(100)));
```