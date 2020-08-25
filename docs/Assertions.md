Assertions are central to unit testing in any of the xUnit frameworks, and NUnit is no exception. NUnit provides a rich set of assertions as static methods of the Assert class.

If an assertion fails, the method call does not return, and an error is reported. If a test contains multiple assertions, any that follow the one that failed will not be executed. For this reason, it's usually best to try for one assertion per test.

NUnit supports two models to write assertions: [Constraint model](https://github.com/nunit/docs/wiki/Constraint-Model) and [Classic model](https://github.com/nunit/docs/wiki/Classic-Model). In the latest versions the Classic model has been reimplemented using the newer Constraint model and will not receive new features. For this reason, we will focus on the Constraint model.

The Constraint model is built around the statement `Assert.That(actualValue, constraint)` where constraint is an object implementing the `IResolveConstraint` interface. Furthermore, NUnit provides many helper methods aiming at creating a fluent expression.

Here are some examples of assertions builts using the constraints model
- `Assert.That(result, Is.EqualTo(42))`
- `Assert.That(result, Is.GreaterThan(0))`
- `Assert.That(result, Is.Not.Null)`
- `Assert.That(result, Is.False)`

`That` supports multiple overloads that allow to specify the message to be returned in case of failure and more.

[Here is a list of all built-in constraints available in NUnit.](https://docs.nunit.org/articles/nunit/writing-tests/constraints/Constraints.html)

## Comparing values
Asserting equality between two values is less intuitive than one would think because several aspects need to be taken in consideration.

By default, the `EqualConstraint`  uses the closest override of the `Object.Equals` method.

When comparing numeric types, developers can use the methods `Within` to specify the tolerance, both in absolute and relative terms. Comparison constraints like `GreaterThan`, `GreaterOrEqualThan`, `LessThan` and `LessOrEqualThan` also expose the `Within` methods.

When comparing strings, developers can use `IgnoreCase` to perform a case-insensitive comparison.

If that is not enough, the developer can specify how the equality check is performed by using the method Using to specify a comparer.

The comparer can be
- An instance of a class implementing `IEquatableComparer` or `IEquatableComparer<T>`
- An instance of a class implementing `IComparer` or `IComparer<T>`
- An instance of a class inheriting from `Comparison<T>`
- A `Func<T, T, bool>` delegate

[Here is the official documentation regarding the EqualConstraint.](https://docs.nunit.org/articles/nunit/writing-tests/constraints/EqualConstraint.html)

Finally, the `SameAsConstraint` checks that the two variables are referencing the same object.

[Here is the official documentation regarding the SameAsConstraint.](https://docs.nunit.org/articles/nunit/writing-tests/constraints/SameAsConstraint.html)

## Expecting exceptions
Sometimes we expect our code to throw an exception under given conditions.

As for normal assertions, NUnit provides several utilities to test whether the system under test throws an exception.

Following the Constraint model showed earlier, developers can use overloads of the `That` method to assert the expected behavior.

The two basic overloads that will be used in this scenario are:
- `That(TestDelegate, IResolveConstraint)`
- `That(AsyncTestDelegate, IResolveConstraint)`

`TestDelegate` and `AsyncTestDelegate` simply represent any synchronous and asynchronous action.

Additionally, NUnit provides helper methods aiming at describing the expected exception in a fluent manner.
Here are some examples:
- `Assert.That(() => sut.DoSomething(null), Throws.ArgumentNullException)`
- `Assert.That(() => sut.DoSomething(""), Throws.ArgumentException)`
- `Assert.That(() => sut.DoSomethingElseAsync(), Throws.TypeOf<MyCustomException>())`

## Combining assertions
Often, the state of the system under test cannot be described in a single statement.

NUnit offers some alternatives to handle this scenario: they mostly overlap but some might not always be applicable and others might not be optimal.
The first option is to stack several calls to the Assert utilities, one after the other.
```csharp
Assert.That(result, Is.Not.Null);
Assert.That(result, Is.EqualTo("Some expected value"));
```
This option is the most intuitive one but its main drawback is that if the first assertion fails, the subsequent ones will not be executed.

The second option is to combine multiple constraints into one using logical operators
```csharp
Assert.That(result, new AndConstraint(
	new NotConstraint(
		new NullConstraint()
	),
	new EqualConstraint("Some expected value")
));
```

The same can also be achieved using the fluent syntax helpers
```csharp
Assert.That(result, Is.Not.Null.And.EqualTo("Some expected value"));
```

The problem with this approach is that the combined constraints can only target the same "actual value". There are ways to explore properties of an object by passing the name of the property as string. Unfortunately, using strings to navigate properties isn't the most optimal approach as there is no check from the compiler and causing the test to fail if the properties were to be renamed in the future.
```csharp
var result = new {IsSuccess = true, Value = 1};
Assert.That(result, Has.Property("IsSuccess").True.And.Property("Value").EqualTo(1));
```
The third option comes in help in this case.
By leveraging `Assert.Multiple`, developers can cluster together multiple assertions with the guarantee that all will be executed even if any of them were to fail.
```csharp
Assert.Multiple(() => 
{
	Assert.That(result.IsSuccess, Is.True);
	Assert.That(result.Value, Is.GreaterThan(0));
});
```
[Here is the official documentation regarding this option.](https://github.com/nunit/docs/wiki/Multiple-Asserts)

## Evaluating collections
Sometimes, tests need to assert the state of collections of items. NUnit includes constraints that help dealing with collections and their items.

When comparing two collections, the following scenarios are supported
- The two collections must contain the same elements in the same order: `Assert.That(actual, Is.EqualTo(expected))`
- The two collections must contain the same elements in any order: `Assert.That(actual, Is.EquivalentTo(expected))`
- One collection is a subset of the other: `Assert.That(actual, Is.SubsetOf(expected))`
- One collection is a superset of the other: `Assert.That(actual, Is.SupersetOf(expected))`
- The tested collection is empty: `Assert.That(actual, Is.Empty)`

Furthermore, the constraints `AllItemsConstraint`, `SomeItemsConstraint` and `NoItemConstraint` allow to make assertions about the items of the collection. 

Like for other constraints, these ones too have fluent equivalents
- All items must match the condition: `Assert.That(collection, Has.All.GreaterThan(0))`
- At least 1 item must match the condition: `Assert.That(collection, Has.Some.GreaterThan(0))`
- No item must match the condition: `Assert.That(collection, Has.None.GreaterThan(0))`
- Exactly the specified number of items must match the condition: `Assert.That(collection, Has.Exactly(3).GreaterThan(0))`

## Custom constraints
Since the second parameter of That can be any object implementing the `IResolveConstraint` interface, developers can create custom constraints and use them in the `That` method
- `Assert.That(result, new MyCustomConstraint(expectedValue))`

Custom constraints should be used to glue NUnit with other libraries or to test specific aspects of the domain.

Custom constraints should be avoided as a mean to combine multiple assertions.

## Assumptions
Assumptions are intended to express the state a test must be in to provide a meaningful result. They are functionally similar to assertions, however a unmet assumption will produce an _Inconclusive_ test result, as opposed to a Failure.

Assumptions make use of the `Assume` static class.
`Assume.That` has the same set of overloads as `Assert.That`. 