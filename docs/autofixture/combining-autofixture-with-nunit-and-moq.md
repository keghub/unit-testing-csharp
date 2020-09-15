Using the glue libraries to combine AutoFixture with [NUnit](./nunit-glue-library.md) and [Moq](./moq-glue-library.md) gives the possibility to write very powerful and semantic-rich unit tests.

To combine these libraries, it's necessary to create a [custom version of the `AutoData` attribute](./nunit-glue-library.md#customizing-the-fixture) that customizes a fixture with the `AutoMoqCustomization`.

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class CustomAutoDataAttribute : AutoDataAttribute
{
    public CustomAutoDataAttribute() : base (CreateFixture) {}

    private IFixture CreateFixture()
    {
        var fixture = new Fixture();

        fixture.Customize(new AutoMoqCustomization { ConfigureMembers = true, GenerateDelegates = true });

        return fixture;
    }
}
```

The `CustomAutoDataAttribute` attribute above shows how to customize a fixture by adding an instance of `AutoMoqCustomization`.

Let's consider the following service as an example

```csharp
public interface IDependency
{
	void SendMessage(string message);
}

public class Service
{
	private readonly IDependency _dependency;
	private readonly ILogger<Service> _logger;

	public Service (IDependency dependency, ILogger<Service> logger)
	{
		_dependency = dependency ?? throw new ArgumentNullException(nameof(dependency));
		_logger = logger ?? throw new ArgumentNullException(nameof(logger));
	}

	public void DoSomething(string message)
	{
		_dependency.SendMessage(message);
	}
}
```

Here is a unit test testing that, when invoking `DoSomething`, the message is forwarded to the `SendMessage` method of the `IDependency`

```csharp
[Test]
public void DoSomething_forwards_message_to_Dependency()
{
	// ARRANGE
	var fixture = new Fixture();
	var mockDependency = new Mock<IDependency>();
	var message = fixture.Create<string>();

	var sut = new Service(mockDependency.Object, Mock.Of<ILogger<IDependency>>());

	// ACT
	sut.DoSomething(message);

	// ASSERT
	mockDependency.Verify(p => p.SendMessage(message), Times.Once());
}
```

The same test can be rewritten using the `CustomAutoDataAttribute` shown above

```csharp
[Test, CustomAutoDataAttribute]
public void DoSomething_forwards_message_to_Dependency([Frozen] IDependency dependency, Service sut, string message)
{
	// ACT
	sut.DoSomething(message);

	// ASSERT
	Mock.Get(dependency).Verify(p => p.SendMessage(message), Times.Once());
}
```

The snippet above shows how well AutoFixture, NUnit and Moq integrate with each other. This is clearly shown by the frozen parameter that gets fed with an implicit mock. `[Frozen] IDependency dependency`