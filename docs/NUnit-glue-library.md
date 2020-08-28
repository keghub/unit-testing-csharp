AutoFixture offers a glue library for NUnit 3.0. In AutoFixture's lingo, a glue library is a library that extend one library (in this case NUnit 3.0) with AutoFixture.

The NUnit glue library is contained in the [`AutoFixture.NUnit3`](https://www.nuget.org/packages/AutoFixture.NUnit3/) NuGet package and is composed by classes that link AutoFixture into NUnit testing pipeline.

## `AutoData` and `InlineAutoData` attributes

The `AutoData` and `InlineAutoData` attributes sit at the center of the integration of AutoFixture with the NUnit pipeline. They take advantage of the same subsystem used to support [parameterized tests](Parameterized-tests) and extend it to feed anonymous objects as test parameters.

### `AutoData` attribute
Here is an example with the `AutoData` attribute.
```csharp
public class Service
{
    public string Echo(string message) => message;
}

[Test]
[AutoData]
public void Echo_returns_same_message(Service sut, string message)
{
    // ACT
    var result = sut.Echo(message);

    // ASSERT
    Assert.That(result, Is.EqualTo(message));
}
```
In the snippet above, it's noticeable that there is no _Arrange_ phase. This is taken care by the `AutoData` attribute used to decoarate the unit test.

Specifically, the `AutoData` attribute took care of
- creating an instance of `Fixture`,
- inspecting the parameters of the unit test
- for each parameter
  - use the fixture instance to generate a value 
  - pass the generated value to NUnit

### `InlineAutoData` attribute
Here is an example with the `InlineAutoData` attribute.
```csharp
public class Service
{
    public int Add(int first, int second) => first + second;
}

[Test]
[InlineAutoData(1)]
[InlineAutoData(1, 10)]
public void Add_returns_sum_of_parameters(int first, int second, Service sut)
{
    // ACT
    var result = sut.Add(first, second);

    // ASSERT
    Assert.That(result, Is.EqualTo(first + second));
}
```

While `AutoData` takes care of providing all parameters, `InlineAutoData` gives the developer the possibility to specify some of the parameters and takes care of generating the ones whose value was not specified. Also, unlike the `AutoData` attribute, the `InlineAutoData` attribute can be applied more than once on the same unit test. Each instance of the attribute will generate a new execution of the unit test.

Here is how the `InlineAutoData` works
- for each instance of the `InlineAutoData`
  - creates an instance of `Fixture`
  - inspects the parameters of the unit test
  - for each parameter
    - if a value was provided in the constructor of the attribute, pass it to NUnit
    - otherwise, use the fixture instance to generate a value and pass it to NUnit

It is important to note that the `InlineAutoData` attribute suffers of the same restrictions of the [`TestCase` attribute](Parameterized-tests#testcase-attribute) regarding which types can be used in the attribute. Unfortunately, unlike the `TestCase` attribute, the `InlineAutoData` attribute doesn't have an equivalent attribute pointing at a method like the `TestCaseSource` attribute.