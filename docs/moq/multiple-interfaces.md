# Multiple interfaces

Classes can implement multiple interfaces. It might happen that the component under test expects the incoming dependency to implement more than one interface, like in the snippet below.

```csharp
public class Dependency : IDependency, IDisposable { ... }

public class Service
{
    private readonly IDependency _dependency;

    public Service (IDependency dependency) { _dependency = dependency; }

    public void DoSomething()
    {
        using (_dependency as IDisposable) { ... }
    }

    public void Dispose() { ... }
}
```

While being a signal that responsibilities are not properly split across interfaces, it's something that developers need to be able to cope with while developing unit tests.

Moq offers a way to decorate a mock with additional interfaces using the construct `As`, giving developers the possibility to configure and verify methods of the added interface.

```csharp
var mock = new Mock<IDependency>();
mock.As<IDisposable>().Setup(p => p.Dispose());
```

Alternatively, you can also store the reference returned by As into its own variable for later usage.

```csharp
var mockDisposable = mock.As<IDisposable>();
mockDisposable.Verify(p => p.Dispose(), Times.Once());
```

