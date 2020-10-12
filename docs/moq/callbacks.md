# Callbacks

A powerful capability of Moq is to attach custom code to configured methods and properties' getters and setters.

This capability is often referred to as Callbacks.

```csharp
var mock = new Mock<IService>();
mock.Setup(p => p.DoSomething()).Callback(() => TestContext.Progress.Writeline("Here"));
```

When configuring methods with arguments, the Callback method can accept two types of delegates.

* A parameterless `Action`

  ```csharp
  mock.Setup(p => p.DoSomethingMore(It.IsAny<int>())).Callback(() => TestContext.Progress.Writeline($"Incoming call"));
  ```

* An `Action` with the same parameters of the configured method. In this case the incoming parameters are forwarded to the given delegate

  ```csharp
  mock.Setup(p => p.DoSomethingMore(It.IsAny<int>())).Callback((int a) => TestContext.Progress.Writeline($"Incoming call: {a}"));
  ```

  For methods without parameters, only the parameterless Action overload is valid. Using invalid overload will cause exceptions at runtime.

## Callbacks and properties

When configuring properties, it's important to remember that properties are just syntactic sugar in front of getter and setter methods. For this reason, callbacks must specifically target those methods.

```csharp
var mock = new Mock<MyAbstractClass>();

mock.SetupGet(p => p.Property)
    .Callback(() => TestContext.Progress.Writeline("Getter invoked"));

mock.SetupSet(p => p.Property = It.IsAny<string>())
    .Callback((string str) => TestContext.Progress.Writeline($"Setter received value: {str}"));
```

Please notice that getter methods have no parameters while setter methods have a single parameter of the type of the property. Like for normal methods, setters can be configured with a parameterless action or with one with a single parameter of the type of the property.

## Altering the state

Callbacks can be used to alter the state of the unit test.

```csharp
int value = 0;
var mock = new Mock<IService>();

mock.Setup(p => p.DoSomething(It.IsAny<int>()))
    .Callback((int x) => value = x);
```

Although possible, it's worth investigating the need for altering the state and if there are better ways to achieve the same result.

The snippet below uses an external variable to count the executions and throwing an exception on the nth invocation

```csharp
int counter = 0;
mock.Setup(p => p.DoSomething()).Callback(() => { if (counter++ >= 5) throw new Exception(); });
```

The same can be achieved with a sequence, making the test easier to read and its intent clearer

```csharp
mock.SetupSequence(p => p.DoSomething())
    .Pass()
    .Pass()
    .Pass()
    .Pass()
    .Pass()
    .Throws<Exception>();
```

## Chaining multiple callbacks

Moq does not support chaining of calls to the Callback method. If multiple actions need to be executed, they need to be called from within the delegate.

```csharp
mock.Setup(p => p.DoSomething(It.IsAny<int>())).Callback((int value) => 
{
    DoSomething();
    DoSomethingWithValue(value)
});
```

