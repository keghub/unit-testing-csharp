# Verifications

When writing unit tests for components consuming dependencies, it's important to make sure the right calls were invoked on the dependencies.

Moq has built-in support for tracking invocations: it leverages the same argument matching syntax used for configuring methods to filter invocations. When inspecting invocation, developers can even assert the expected count.

The snippets in this section will be based on the following interface

```csharp
public interface IService 
{
    void Send(string message);

    Task SendAsync(string message);

    event EventHandler<MessageEventArgs> Sent;

    string ContentType { get; set; }
}

public class MessageEventArgs : EventArgs
{
    public string Message { get; set; }
}
```

## Implicit verification

The easiest way to verify that the calls configured on mocks were actually performed is by using the `Verifiable` construct at the end of each configuration.

```csharp
mock.Setup(p => p.Send(It.IsAny<string>())).Verifiable();
```

Later, we can use the `VerifiyAll` to verify that all configurations enriched with `Verifiable` were matched by an invocation.

```csharp
mock.VerifyAll();
```

Optionally, developers can customize the fail message outputted by Moq if the verification were to fail.

```csharp
mock.Setup(p => p.Send(It.IsAny<string>())).Verifiable("Send was never invoked");
```

If your unit test uses multiple mocks, you can use `Mock.Verify` to verify all verifiable configuration at once.

```csharp
Mock.Verify(mock, anotherMock, yetAnotherMock);
```

## Explicit verification

Another approach is to explicitly verify that a certain invocation was performed. While more verbose, it offers more powerful tools to developers.

```csharp
mock.Setup(p => p.Send(It.IsAny<string>()));
mock.Verify(p => p.Send(It.IsAny<string>()));
```

The `Verify` method offers additional overloads to specify the fail message and the amount of times the invocation was expected to be performed. The amount of times can be specified as a known value or via a lazily evaluated expression.

```csharp
mock.Verify(p => p.Send(It.IsAny<string>()), "Send was never invoked");
mock.Verify(p => p.Send(It.IsAny<string>()), Times.AtLeastOnce());
mock.Verify(p => p.Send(It.IsAny<string>()), Times.AtLeastOnce(), "Send was never invoked");
mock.Verify(p => p.Send(It.IsAny<string>()), () => Times.AtLeastOnce(), "Send was never invoked");
```

Note that since `Verify` accepts both `Times` and `Func<Times>`, both `Times.AtLeastOnce()` and `Times.AtLeastOnce` are accepted by the compiler.

## Times

The `Times` class exposes several factory methods to define the constraint on amount of invocations as needed. The basic methods are

* `Exactly`: the constraint will fail if the configuration isn't used the specified amount of times

  ```csharp
  Times.Exactly(3)
  ```

* `AtLeast`: the constraint will fail if the configuration is used for a number of time less than the specified amount

  ```csharp
  Times.AtLeast(3)
  ```

* `AtMost`: the constraint will fail if the configuration is used for a number of time greater than the specified amount

  ```csharp
  Times.AtMost(3)
  ```

* `Between`: the constraint will fail if the configuration is used a number of times outside of the specified interval

  ```csharp
  Times.Between(3, 5, Range.Inclusive)
  Times.Between(2, 4, Range.Exclusive)
  ```

  On top of these methods, there are other helpers that can be used to streamline the creation of constraints

* `Never`: the constraint will fail if the configuration is used at least once

  ```csharp
  Times.Never()
  ```

* `Once`: the constraint will fail if the configuration is not used or used more than once

  ```csharp
  Times.Once()
  ```

* `AtLeastOnce`: the constraint will fail if the configuration is never used

  ```csharp
  Times.AtLeastOnce()
  ```

* `AtMostOnce`: the constraint will fail if the configuration is used more than once. Note that no usage is also valid for the constraint.

  ```csharp
  Times.AtMostOnce()
  ```

## Verifying calls on properties

Like for methods, Moq supports the verification of matching invocations for properties. Developers can configure implicit verification

```csharp
mock.SetupProperty(p => p.ContentType, "text/plain").Verifiable();
mock.SetupGet(p => p.ContentType).Returns("text/plain").Verifiable();
mock.SetupSet(p => p.ContentType = It.IsAny<string>()).Verifiable();
```

Alternatively, developers can explicitly verify the performed invocations.

```csharp
mock.VerifyGet(p => p.ContentType, Times.Once());
mock.VerifySet(p => p.ContentType = "text/plain", Times.Once());
```

Finally, `Verifiable`, `VerifyGet` and `VerifySet` offer an overload to specify a custom message to return in case the constraint fails.

## Implicit or explicit verification?

Moq supporting two different styles of verification makes room for making unit tests either unnecessarily tight or dangerously open. The agency offered by Moq consolidates in two approaches

* Open configuration, close verification
* Close configuration, open verification

When following the first approach, developers should configure methods and properties relying as much as possible on constructs like `It.IsAny`. In the _Assert_ section of the unit test, developers should then rely on explicit verification with constraints as specified as possible.

```csharp
mock.Setup(p => p.Send(It.IsAny<string>()));
mock.Verify(p => p.Send("Hello world"), Times.Once());
```

On the other hand, when following the second approach, developers should configure methods and properties specifying as much as possible the expected parameters and rely on implicit verification.

```csharp
mock.Setup(p => p.Send("Hello world").Verifiable();
mock.VerifyAll();
```

Please note that the two approaches are not exactly equivalent as implicit verification does not support checking the amount of invocations.

As mentioned above, using the remaining two approaches \(open/open and close/close\) is suboptimal when not dangerous. The open/open approach below makes the verification almost useless.

```csharp
mock.Setup(p => p.Send(It.IsAny<string>())).Verifiable();
mock.VerifyAll();
```

The close/close approach below makes writing the unit test too cumbersome for no real gain

```csharp
mock.Setup(p => p.Send("Hello world"));
mock.Verify(p => p.Send("Hello world"), Times.Once());
```

## Verifying event handling

When dealing with events, it's important to make sure that events are properly handed both to ensure the component works as expected.

```csharp
service.Sent += MessageSent;
```

Similarly, registered events should be unregistered to avoid annoying memory leaks.

```csharp
service.Sent -= MessageSent;
```

Moq offers the possibility to set expectations on both registration and unregistration of event handler.

```csharp
mock.VerifyAdd(p => p.Sent += It.IsAny<EventHandler<MessageEventArgs>>());
mock.VerifyRemove(p => p.Sent -= It.IsAny<EventHandler<MessageEventArgs>>());
```

