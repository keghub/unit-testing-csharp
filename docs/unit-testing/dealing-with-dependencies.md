# Dealing with dependencies

Most components rely on dependencies to perform their tasks. Delegating certain concerns to dependencies makes it easier to grasp the intent of each component and focus on their responsibilities rather than being distracted by implementation details.

On the other hand, dependencies require special attention when testing the expected behavior of each component. The main concern is that unit tests should be focusing on the system under test and not leak into testing the behavior of its dependencies. Therefore, it's important to find a way to exclude concrete dependencies when running the tests.

Usually, following best practices like the SOLID principles goes a long way into making a codebase easily testable. To be more precise, the Dependency Inversion Principle and the Dependency Injection techniques help designing components whose dependencies can be easily replaced.

When dealing with a codebase built following the SOLID principles, developers can rely on two different techniques: stubs and mocks. Fakes \(or stubs\) are ad-hoc implementations of the dependencies' interfaces that can be passed when instantiating the system under test. While keeping the shape of the dependency, the fakes will not behave like the real dependencies.

Let's take this service as example: its constructor requires a dependency of type `IDependency`.

```csharp
public interface IDependency
{
    void DoSomething(string value);
}

public class MyService
{ 
    private readonly IDependency _dependency;

    public MyService (IDependency dependency)
    {
        _dependency = dependency ?? throw new ArgumentNullException(nameof(dependency));
    }

    public void SendCommand(string command)
    {
        _dependency.DoSomething(command);
    }
}
```

When setting up a unit test for the method `SendCommand`, we need an object implementing `IDependency` to create an instance of `MyService`. For this reason, we can create a fake of `IDependency`. Additionally, we can introduce custom logic to track the input it receives and how many times it gets executed.

```csharp
public class FakeDependency : IDependency
{
    public void DoSomething(string value)
    {
        InputValue = value;
        ExecutionCount++;
    }

    public int ExecutionCount { get; private set; }

    public string InputValue { get; private set; }
}
```

With this fake dependency in place, we can now create a unit test that passes an arbitrary string to the component under test and expects that the same string is forwarded to the underlying dependency.

```csharp
[Test]
public void SendCommand_forwards_commands_to_dependency()
{
    // ARRANGE
    const string command = "TEST_COMMAND";
    var dependency = new FakeDependency();
    var sut = new MyService(dependency);

    // ACT
    sut.SendCommand(command);

    // ASSERT
    Assert.That(dependency.InputValue, Is.EqualTo(command));
}
```

While working, this approach is less than optimal because it forces the developer to create multiple fake dependencies to cover all cases. In the example, what if we wanted to simulate our dependency throwing an exception? In this case, we would need to create another class. Mocking frameworks exist to avoid this problem. Rather than create a new class for every case, they use an API to let the developer describe the behavior of the dependency.

```csharp
[Test]
public void SendCommand_forwards_commands_to_dependency()
{
    // ARRANGE
    const string command = "TEST_COMMAND";
    var dependency = Mock.Of<IDependency>();
    var sut = new MyService(dependency);

    // ACT
    sut.SendCommand(command);

    // ASSERT
    Mock.Get(dependency).Verify(p => p.DoSomething(command), Times.Once());
}
```

As you can see in the snippet above, the test doesn't require an extra class as the mocking framework used in the test lets us describe the intended behavior for the dependency.

