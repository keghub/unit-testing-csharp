# Protected members

Unlike interfaces, classes \(abstract and non\) can have members that can be accessed only by their inheritors. These members are often defined "protected members" as for the visibility flag they are decorated with. When testing components accepting classes with these members, developers might need to configure methods not accessible to them.

Moq supports two ways to configure protected members. Unfortunately, none of them can ensure type safety being based on string and mapping types.

The snippets in this section will be based on the following class

```csharp
public abstract class ServiceBase
{
    protected abstract void Step1();

    protected abstract void Step2(string value);

    protected abstract int Count { get; set; }

    public void DoSomething(string value)
    {
        Step1();
        Step2(value);
    }
}
```

Finally, please note that both approaches require importing of the `Moq.Protected` namespace. Also, the helpers targeting protected members are accessible via the `Protected()` construct.

## Configuring members by name

The `Protected` construct offers helpers to configure members via their name.

```csharp
mock.Protected()
    .Setup("Step1")
    .Callback(() => Debug.WriteLine("Step1"));
```

Parameters can be specified directly or via argument matching: unlike normal methods, developers should use `ItExpr` instead of `It`.

```csharp
mock.Protected()
    .Setup("Step2", ItExpr.IsAny<string>())
    .Callback((string value) => Debug.WriteLine($"Step2: {value}"));
```

Unable to rely on static typing, Moq requires explicit typing when configuring properties.

```csharp
mock.Protected()
    .SetupGet<int>("Count")
    .Returns(42);

mock.Protected()
    .SetupSet<int>("Count", ItExpr.IsAny<int>())
    .Callback((int value) => Debug.WriteLine($"Count set to {value}"));
```

## Configuring members using a mapping type

Configuring members by name and without explicit typing can be avoided by instructing Moq to use a mapping type as reference.

The following interface mimics the protected methods exposed by the `ServiceBase` class.

```csharp
public interface IServiceBaseMapping
{
    void Step1();
    void Step2(string value);
    int Count { get; set; }
}
```

Once this type is declared, it's possible to use more comfortable methods to configure the protected methods.

```text
mock.Protected()
    .As<IServiceBaseMapping>()
    .Setup(p => p.Step1())
    .Callback(() => Debug.WriteLine("Step1"));
```

Parameters can be defined using `It` as usual.

```csharp
mock.Protected()
    .As<IServiceBaseMapping>()
    .Setup(p => p.Step2(It.IsAny<string>()))
    .Callback((string value) => Debug.WriteLine($"Step2: {value}"));
```

Also properties are more comfortable to configure when using a mapping type

```csharp
mock.Protected()
    .As<IServiceBaseMapping>()
    .SetupProperty(p => p.Count, 42);
```

