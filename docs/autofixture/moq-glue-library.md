Like for NUnit, AutoFixture offers a glue library for Moq.

By using this library, AutoFixture can use Moq to handle the requests for non-concrete types like abstract classes and interfaces. Optionally, AutoFixture can delegate to Moq the creation of [fake delegates](../moq/delegates.md).

The library is contained in the [`AutoFixture.AutoMoq`](https://www.nuget.org/packages/AutoFixture.AutoMoq/) NuGet package.

The `AutoMoqCustomization` is the core of the integration of AutoFixture with Moq. By adding an instance of this class to the fixture, requests for non-concrete types will be handled by Moq.

```csharp
var fixture = new Fixture();
fixture.Customize(new AutoMoqCustomization());
```

The snippets below are based on the following types

```csharp
public class Service
{
    public Service(IDependency dependency, AbstractType innerObject)
    {
        Dependency = dependency ?? throw new ArgumentNullException(nameof(dependency));
        InnerObject = innerObject ?? throw new ArgumentNullException(nameof(innerObject));
    }

    public IDependency Dependency { get; }
    
    public AbstractType InnerObject { get; }
}

public interface IDependency
{
    void DoSomething(Action<string> action) { action("Hello world"); }
}

public abstract class AbstractType
{
    public int IntValue { get; set; }
    
    public string StringValue { get; set; }
}
```

## Requesting objects

Once a fixture is enriched with `AutoMoqCustomization`, developers can use it to request anonymous instances of non-concrete types.

This can be done in these ways.
- By requesting directly the type to be mocked. In this case an [implicit mock](../moq/implicit-mocks.md) will be returned.
- By requesting for a mock of the type. In this case an instance of `Mock<T>` will be returned.

It is important to note that [_freezing_](./type-customization.md#freeze) is supported in both cases.

```csharp
[Test]
public void Freezing_is_supported_when_requesting_type()
{
    // ARRANGE
    fixture.Customize(new AutoMoqCustomization());
    var dependency = fixture.Freeze<IDependency>(); // freezing the type directly

    // ACT
    var sut = fixture.Create<Service>();

    // ASSERT
    Assert.That(sut.Dependency, Is.SameAs(dependency));
}

[Test]
public void Freezing_is_supported_when_requesting_Mock_of_type()
{
    // ARRANGE
    fixture.Customize(new AutoMoqCustomization());
    var mockDependency = fixture.Freeze<Mock<IDependency>>(); // freezing a mock of the type

    // ACT
    var sut = fixture.Create<Service>();

    // ASSERT
    Assert.That(sut.Dependency, Is.SameAs(mockDependency.Object));
}
```

## Mock configuration

Mocks requested by AutoFixture via AutoMoq have their configuration that differs from the normal defaults of Moq.

Specifically, here is the list of the properties of the mocks
- The [`CallBase` property](../moq/base-class.md) is set to `true`
- The [`MockBehavior` property](../moq/mock-customization.md#mock-behavior) is set to `Loose`
- The [`DefaultValue` property](../moq/mock-customization.md#default-value) is set to `Mock`

## `ConfigureMembers`

By default, AutoMoq only provides values for non-concrete types.

```csharp
[Test]
public void AutoMoq_does_not_provide_values_by_default()
{
    // ARRANGE
    fixture.Customize(new AutoMoqCustomization());

    // ACT
    var sut = fixture.Create<Service>();

    // ASSERT
    Assert.That(sut.Dependency, Is.Not.Null);
    Assert.That(sut.InnerObject, Is.Not.Null);
    Assert.That(sut.InnerObject.StringValue, Is.Null);
}
```

This can be overridden by setting the property `ConfigureMembers` to `true`.

```csharp
[Test]
public void AutoMoq_provides_values_if_configured()
{
    // ARRANGE
    fixture.Customize(new AutoMoqCustomization { ConfigureMembers = true });

    // ACT
    var sut = fixture.Create<Service>();

    // ASSERT
    Assert.That(sut.Dependency, Is.Not.Null);
    Assert.That(sut.InnerObject, Is.Not.Null);
    Assert.That(sut.InnerObject.StringValue, Is.Not.Null);
}
```

## `GenerateDelegates`

By setting the property `GenerateDelegates` to `true`, developers can instruct AutoFixture to delegate the creation of delegates to AutoMoq.

```csharp
[Test]
public void DoSomething_uses_given_action()
{
    // ARRANGE
    fixture.Customize(new AutoMoqCustomization { GenerateDelegates = true });
    var action = fixture.Create<Action<string>>();
    var sut = fixture.Create<Service>();

    // ACT
    sut.Dependency.DoSomething(action);

    // ASSERT
    Mock.Get(action).Verify(p => p("Hello world"));
}
```

Like non-concrete types, delegates can be requested directly by their type or by requesting a mock of the type.
