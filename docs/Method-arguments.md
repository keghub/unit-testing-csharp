When configuring method calls, it's important to set proper expectations on the incoming arguments. Developers can set expectations on incoming parameters for each method call setup. If the expectation on the parameter are not matched, the setup will not be used for the specific call.

## Any instance of the parameter type
The most basilar expectation accepts all instances of the argument type. The following snippet configure the mock to accept calls of the `DoSomething` method regardless of the incoming argument. 
```csharp
mock.Setup(p => p.CreateUser(It.IsAny<string>()));
```
Note: being Moq a type-safe framework, it's impossible to provide parameters that don't match with the method signature.

## Specific instance of the parameter type
On the opposite side of the spectrum, developers can specify that the call setup should be used only if the incoming parameter is equal to the one specified in the setup.
```
mock.Setup(p => p.CreateUser("TEST_USERNAME"));
```
Note that general rules for equality comparison apply: strings and value types will be compared by their actual value, reference types will be compared on their reference unless the `Object.Equals` has been properly overridden.

## Arguments matching certain patterns
Somewhere in between "any instance" and "specific instance" there are many patterns used to compare expected arguments and actual incoming values.

`It.IsRegex` only accepts incoming strings matching the given regular expression pattern
```csharp
mock.Setup(p => p.CreateUser(It.IsRegex("[A-Z_]+")));
```

`It.IsInRange` only accepts numbers between the two extremes of the given range
```csharp
mock.Setup(p => p.GetUserById(It.IsInRange(0, 100, Moq.Range.Exclusive)));
```

`It.Is` only accepts parameters that are positively evaluated by the given predicate
```csharp
mock.Setup(p => p.CreateUser(It.Is<string>(s => s.StartsWith("TEST"))));
```
Note that the predicate is evaluated only when the method is actually invoked and the `s` variable represents the incoming parameter.