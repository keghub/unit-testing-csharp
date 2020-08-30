# Results

When configuring mocks, it is important to specify the return value of functions \(methods that return a value\) and properties.

Moq supports this scenario with the `Returns` construct.

As previously seen, the most elementary scenario is configuring a method or a property to return a well-known value, i.e. a value already available while configuring the mock.

```csharp
var mock = new Mock<MyAbstractClass>();
mock.Setup(p => p.Property).Returns("Hello world");
mock.Setup(p => p.GetRandomNumber()).Returns(123);
```

Note that the return value must match with the return type of the the configured member.

## Asynchronous functions

Moq exposes helper methods to deal with asynchronous functions \(i.e. methods returning Task\).

In the snippet below, the two lines are interchangeable, with the clear advantage of letting Moq handling the Task API.

```csharp
mock.Setup(p => p.GetValueAsync()).Returns(Task.FromResult(123));
mock.Setup(p => p.GetValueAsync()).ReturnsAsync(123);
```

## Computed return values

While configuring the mock, there might be the need for computing the return value based on the incoming inputs. To handle this scenario, `Returns` and `ReturnsAsync` have a set of overloads accepting a delegate instead of a finite value.

```csharp
mock.Setup(p => p.Add(It.IsAny<int>(), It.IsAny<int>())
    .Returns((int first, int second) => first + second);

mock.Setup(p => p.AddAsync(It.IsAny<int>(), It.IsAny<int>())
    .ReturnsAsync((int first, int second) => first + second);
```

Note that the delegate must match the method signature and will be lazily invoked when the method is invoked.

## Returning `null`

Due to how overload resolution works in C\#, configuring a method or a property to return a `null` value requires some attention.

In fact, simply typing `.Returns(null)` will cause a compiler error:

> CS0121 The call is ambiguous between the following methods or properties: 'IReturns.Returns\(TResult\)' and 'IReturns.Returns\(Func\)'

To solve this, it's enough to cast `null` to the target type.

```csharp
var mock = new Mock<MyAbstractClass>();
mock.Setup(p => p.Property).Returns(null as string);
```

