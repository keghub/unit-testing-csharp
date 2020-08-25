Once a much more prominent feature of the .NET languages, events are not so commonly used anymore. Despite the fall of grace, many libraries, legacy and not, still use this feature.

Moq has some support for events, especially when following [established patterns](https://docs.microsoft.com/en-us/dotnet/csharp/event-pattern), like using event arguments inheriting from the `EventArgs` class. Whilst there is some support for so-called custom events, they will not be part of this guide. Please refer to the [Moq Quick Start](https://github.com/Moq/moq4/wiki/Quickstart#events) section related to events for this scenario. 

The snippets of this section are based on the following interface
```csharp
public class MessageEventArgs : EventArgs
{
    public string Message { get; set; }
}

public interface IService 
{
    event EventHandler<MessageEventArgs> Sent;

    event EventHandler<MessageEventArgs> Received;

    Task SendAsync(string message);

    void Send(string message);

    string Receive();

    Task<string> ReceiveAsync();
}
```

## Reacting to method calls
One of the most common scenarios is raising an event upon the invocation of a configured method. This can be done by using the `Raises` construct.
```csharp
var mock = new Mock<IService>();
mock.Setup(p => p.Send(It.IsAny<string>()))
    .Raises(e => e.Sent += null, (string msg) => new MessageEventArgs { Message = msg });
```
Usually, developers would use the Raises construct when the component under test relies on the method being invoked upon a method call.
To test the setup to be correct, we can operate on the service as if we were the component under test. We can do this by using the `Object` property of the mock.
```csharp
var service = mock.Object;
```
We then attach an event handler to the event exposed by the interface
```csharp
service.Sent += (object sender, MessageEventArgs args) => Debug.WriteLine(args.Message);
```
Finally, we invoke the `Send` method by passing any string.
```csharp
service.Send("Hello World");
```
If the setup is correct, we should see the `"Hello world"` string in the output stream.

## Reacting to function calls
Like void methods, i.e. methods without return type, Moq can configure functions to raise an event when they are invoked.

The only difference is the requirement for configuring what the method will return before configuring the event `Raise`.
```csharp
mock.Setup(p => p.Receive())
    .Returns("Hello")
    .Raises(e => e.Received += null, (string msg) => new MessageEventArgs { Message = msg });
```

## Reacting to asynchronous methods
In the case we're dealing with asynchronous methods, few adjustments to the snippet above are required.

The rationale behind the adjustments is that asynchronous methods are normal methods returning a `Task`. Moq's strong type system does not make a strong distinction between asynchronous methods and normal methods with a return type.
```csharp
mock.Setup(p => p.SendAsync(It.IsAny<string>()))
    .Returns(Task.CompletedTask)
    .Raises(e => e.Sent += null, (string message) => new MessageEventArgs { Message = message });

mock.Setup(p => p.ReceiveAsync())
    .ReturnsAsync("Hello")
    .Raises(e => e.Received += null, (string msg) => new MessageEventArgs { Message = msg });
```

## Raising events manually
Another use case Moq is able to fulfill is the ability to manually raise events. The scenario is the one where the system under test is accepting a service that expose an event, subscribes to this event and reacts when it is raised. 
```csharp
mock.Raise(p => p.Sent += null, new MessageEventArgs { Message = "Hello world" });
```
This capability is normally used in the _Act_ part of a properly written unit test.