# Method calls

Most of the times, Moq will be used to mock service interfaces and their methods. Moq offers several utilities to properly configure method calls.

As shown earlier, methods can be configured using the Setup method. Additionally, developers can configure sequences of calls. Finally, Moq supports the configuration of methods with less common arguments such as reference parameters, out parameters and optional arguments.

## Single calls

Using the `Setup` method, developers can configure method calls on mocks by leveraging the different argument match techniques shown earlier.

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<string>()));
```

When a mock is configured like above, it will react to any call matching the incoming parameters \(any string, in the example\).

`Setup` also supports asynchronous methods.

```csharp
mock.Setup(p => p.DoSomethingAsync(It.IsAny<string>()));
```

## Repeated calls

Sometimes, we need to configure the mocks so that their methods have different outcomes when called repeatedly.

```csharp
mock.SetupSequence(p => p.GetSomething(It.IsAny<string>())
    .Returns(1)
    .Returns(2)
    .Returns(3);
```

When a mock is configured like above, repeated calls will receive the specified result in sequence. \(`Returns` will be explained more in details later\).

In case the configured method has no return type, Pass can be used to configure an uneventful invocation.

```csharp
mock.SetupSequence(p => p.DoSomething(It.IsAny<string>()))
    .Pass()
    .Pass()
    .Pass();
```

## Calls in sequence

Another scenario supported by Moq is the one of setting up calls in sequence, across different methods and even across different mocks. To be able to properly verify the calling sequence, it's best to set the mocks using the strict mode, more information on this in the advanced section of this guide.

To properly configure sequences, developers need to create an instance of MockSequence and bind the mocks to it.

In the first example we are configuring the mock of a service to respond to calls to a method with an exact sequence of parameters

```csharp
var mockFirst = new Mock<IFirstService>(MockBehavior.Strict);

var sequence = new MockSequence();

mockFirst.InSequence(sequence).Setup(p => p.DoWithInteger(1));
mockFirst.InSequence(sequence).Setup(p => p.DoWithInteger(2));
```

In the second example, we're configuring a sequence of calls of different methods belonging to the same mock

```csharp
var mockFirst = new Mock<IFirstService>(MockBehavior.Strict);

var sequence = new MockSequence();

mockFirst.InSequence(sequence).Setup(p => p.DoWithString(It.IsAny<string>()));
mockFirst.InSequence(sequence).Setup(p => p.DoWithInteger(It.IsAny<int>()));
```

Finally, in the third example, we're configuring a sequence of calls across different mocks

```csharp
var mockFirst = new Mock<IFirstService>(MockBehavior.Strict);
var mockSecond = new Mock<ISecondService>(MockBehavior.Strict);

var sequence = new MockSequence();

mockFirst.InSequence(sequence).Setup(p => p.DoWithInteger(1));
mockSecond.InSequence(sequence).Setup(p => p.DoWithDate(DateTime.Today));
```

When calls are configured within a sequence, Moq will not recognize calls out of sequence and, if operating in strict mode, will throw exceptions for non-configured calls.

Please notice how all mocks are created specifying the parameter `MockBehavior.Strict`. More information on the concept of [mock behavior](./mock-customization.md#mock-behavior) later.

## Parameters passed by reference

Normally, when invoking a method, references to objects are passed by value. This means that the method cannot change the target of the incoming variable. Sometimes this behavior is not desirable and for this reason developers can pass object references by reference to methods. This is achieved by using the `ref` keyword when declaring your method's arguments. [Here is a page on the official documentation regarding passing reference-type parameters by reference](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/passing-reference-type-parameters).

Moq gives the developer the possibility to easily work with "ref parameters". Developers can specify that the method should respond to a specific instance

```csharp
mock.Setup(p => p.DoSomething(ref It.Ref<string>.IsAny));
```

Alternatively, they can specify that a specific instance must be passed to the method, and returned to the caller.

```csharp
var value = "This is a test value";
mock.Setup(p => p.DoSomething(ref value));
```

Finally, although requiring some extra work, developers can customize the referenced object after the method is invoked.

```csharp
delegate void DoSomethingCallback(ref string value);

var newValue = "This is the new referenced value";
mock.Setup(p => p.DoSomething(It.Ref<string>.IsAny))
    .Callback(new DoSomethingCallback((ref string value) => value = newValue));
```

When `DoSomething` is invoked, the variable holding the reference to the passed argument will be holding a reference to the `newValue` variable.

The setup is a bit more convoluted here. The reason is that C\# doesn't support `Action` and `Func` delegates with ref parameters: to obviate this issue, we define a custom delegate and we use it to create a callback function that Moq will invoke when the method is invoked. We will look at how Moq supports callbacks later.

## Out parameters

Similarly to ref parameters, out parameters are used to pass values by reference. [Here is a page on the official documentation regarding out parameters](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier).

Developers can specify that a specific instance must be returned to the caller

```csharp
var value = "This is the expected out value";
mock.Setup(p => p.DoSomething(out value));
```

When `DoSomething` is invoked, the out parameter will be a reference to the preset variable: in the example, the variable `value`.

## Optional arguments

Optional arguments were introduced in C\# 4.0. This functionality gives developers the possibility to mark some parameters of a method as optional by providing a default value. This means that the same method can be invoked with a varying list of parameters.

In the example below, the method DoSomething can be invoked either by passing any string or by leveraging the given default value.

```csharp
public void DoSomething(string operation = "DEFAULT_OPERATION") { }

// call with custom value
service.DoSomething("MY_CUSTOM_OPERATION");

// call with default value
service.DoSomething();
```

When mocking interfaces or classes that expose methods with optional arguments, it's important to remember to specify a value for the optional arguments.

Probably the most common case is the `CancellationToken` to be provided to most asynchronous methods. As most libraries specify a default value for the `CancellationToken` argument, developers often overlook the necessity to specify a value for that argument. In this case developers can either use the `IsAny` construct or the `default` keyword.

```csharp
mock.Setup(p => p.DoSomethingAsync(It.IsAny<string>(), It.IsAny<CancellationToken>()));
mock.Setup(p => p.DoSomethingAsync(It.IsAny<string>(), default));
```

Please note that in the first case, we are accepting any `CancellationToken` while in the latter, we're specifying a specific value. If the component under test were not to rely on the default value, Moq would not capture the invocation.

