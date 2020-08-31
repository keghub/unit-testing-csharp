AutoFixture offers a glue library for NUnit 3.0. In AutoFixture's lingo, a glue library is a library that extend one library (in this case NUnit 3.0) with AutoFixture.

The NUnit glue library is contained in the [`AutoFixture.NUnit3`](https://www.nuget.org/packages/AutoFixture.NUnit3/) NuGet package and is composed by classes that link AutoFixture into NUnit testing pipeline.

The `AutoData` and `InlineAutoData` attributes sit at the center of the integration of AutoFixture with the NUnit pipeline. They take advantage of the same subsystem used to support [parameterized tests](Parameterized-tests) and extend it to feed anonymous objects as test parameters.

## `AutoData` attribute
The `AutoData` attribute is used to automatically generate values to be passed to the unit test, effectively making unit test authoring much faster.

Here is an example of a test written without taking advantage of the `AutoData` attribute.


```csharp
public class Service
{
    public string Echo(string message) => message;
}

[Test]
public void Echo_returns_same_message()
{
    // ARRANGE
    var fixture = new Fixture();
    var sut = fixture.Create<Service>();
    var message = fixture.Create<string>();

    // ACT
    var result = sut.Echo(message);

    // ASSERT
    Assert.That(result, Is.EqualTo(message));
}
```

By using the `AutoData` attribute, the unit test above can be converted as follows.

```csharp
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
There are a few noticeable changes between the two snippets:
- There is no _Arrange_ phase. 
- The `AutoData` attribute decorator has been added the unit test.
- There are new parameters passed to the unit test. 

In fact, all of these changes are connected. By adding the `AutoData` attribute we have effectively moved the _Arrange_ phase out of the unit test and we now accept all the pieces we need to run the test as parameters.

The main advantages of doing so are:
- shorter tests that are easier to scan and understand
- focus on the _Act_ and _Assert_ phases rather than on _Arrange_
- centralized configuration on how to create objects (see below)

Specifically, the `AutoData` attribute took care of
- creating an instance of `Fixture`,
- inspecting the parameters of the unit test
- for each parameter
  - use the fixture instance to generate a value 
  - pass the generated value to NUnit


## `InlineAutoData` attribute
While `AutoData` takes care of providing all parameters, `InlineAutoData` gives the developer the possibility to specify some of the parameters and takes care of generating the ones whose value was not specified.

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
As shown above, unlike the `AutoData` attribute, the `InlineAutoData` attribute can be applied more than once on the same unit test. Each instance of the attribute will generate a new execution of the unit test.

Specifically, the following executions will be performed by NUnit
- `Add_returns_sum_of_parameters(1, <int>, <Service>)`
- `Add_returns_sum_of_parameters(1, 10, <Service>)`

Here is how the `InlineAutoData` works
- for each instance of the `InlineAutoData`
  - creates an instance of `Fixture`
  - inspects the parameters of the unit test
  - for each parameter
    - if a value was provided in the constructor of the attribute, pass it to NUnit
    - otherwise, use the fixture instance to generate a value and pass it to NUnit

It is important to note that the `InlineAutoData` attribute suffers of the same restrictions of the [`TestCase` attribute](Parameterized-tests#testcase-attribute) regarding which types can be used in the attribute. Unfortunately, unlike the `TestCase` attribute, the `InlineAutoData` attribute doesn't have an equivalent attribute pointing at a method like the `TestCaseSource` attribute.

## `Frozen` attribute

As shown above, the `AutoData` and `InlineAutoData` can be delegated the creation of the system under test. Doing so creates an interesting challenge if the system under test accepts dependencies that need to be configured or used during the _Assert_ phase.

Let's take the following class as test subject and assume we want to rewrite the unit test using the `AutoData` attribute.
```csharp
public class Service
{
    private readonly IList<string> _items;
    
    public Service(IList<string> items)
    {
        _items = items ?? throw new ArgumentNullException(nameof(items));
    }
    
    public void Add(string item) => _items.Add(item ?? throw new ArgumentNullException(nameof(item)));
}

[Test]
public void Add_should_add_item_to_underlying_list()
{
    // ARRANGE
    var fixture = new Fixture();
    var list = fixture.Create<List<string>>();
    var sut = new Service(list);
    var item = fixture.Create<string>();

    // ACT
    sut.Add(item);

    // ASSERT
    Assert.That(list, Contains.Item(item));
}
```

Unfortunately, simply decorating the test with the `AutoData` attribute and convert the variables `list`, `sut` and `item` as parameters will not work because the list served as parameter and the one served to the constructor of `Service` will not be the same instance.

The `Frozen` article solves this issue. By leveraging the [`Freeze`](Type-customization#freeze) extension method, it creates an instance of a given type and it uses it to serve successive requests. This attribute is used by decorating which parameter of the unit test needs to be generated using the `Freeze` method and it works with test decorated by both the `AutoData` and `InlineAutoData` attributes.

With the help of the `Frozen` attribute, we can rewrite the test as follow

```csharp
[Test, AutoData]
public void Add_should_add_item_to_underlying_list([Frozen] IList<string> list, Service sut, string item)
{
    // ACT
    sut.Add(item);

    // ASSERT
    Assert.That(list, Contains.Item(item));
}
```

Please notice the sequence of parameters of the rewritten unit test: the frozen parameters must go before the class using them.

## `Greedy` and `Modest` attributes

Another issue deriving from delegating AutoFixture of instantiating classes like system under tests is the loss of control on which constructor is picked. By default, AutoFixture picks the constructor with least parameters, but sometimes this is not the optimal choice.

The `Greedy` and `Modest` attributes give the developer the power to instruct AutoFixture which constructor to select when constructing an object to be passed to NUnit. The `Greedy` attribute will instruct AutoFixture to use the constructor with the most parameters while the `Modest` attribute will instruct AutoFixture to follow the default strategy and pick the constructor with least arguments. In both cases, copy constructors (constructors accepting the same type as single parameter) will be ignored.

The `Greedy` attribute comes in hand when a service exposes a parameterless constructor with defaults that are not suited for testing.

Here is an example showing when to use the `Greedy` attribute.

```csharp
public class ServiceOptions
{
    public string Value { get; set; }
}

public class Service
{
    private readonly ServiceOptions _options;

    public Service () : this (new ServiceOptions()) { }

    public Service (ServiceOptions options) 
    {
        _options = options ?? throw new ArgumentNullException(nameof(options));
    }

    public string GetOptionValue() => _options.Value;
}

[Test, AutoData]
public void GetOptionValue_returns_passed_value([Frozen] ServiceOptions options, [Greedy] Service sut)
{
    // ACT
    var actual = sut.GetOptionValue();

    // ASSERT
    Assert.That(actual, Is.EqualTo(options.Value));
}
```

If neither `Greedy` and `Modest` are able to select the desired constructor, developers will have to instruct AutoFixture via a customization.

## Customizing the fixture

Because the `AutoData` and `InlineAutoData` attributes encapsulate the generation of parameter values, the test author does not need to use an instance of `Fixture` directly making test authoring for common cases quick and trivial.

Unfortunately, doing so prevents the developer from registering customizations and behaviors.

This can be worked around by subclassing the `AutoData` and `InlineAutoData` attributes.

Here is an example of a basic customization of the two attributes.

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class CustomAutoDataAttribute : AutoDataAttribute
{
    public CustomAutoDataAttribute() : base (CreateFixture) {}

    private IFixture CreateFixture()
    {
        var fixture = new Fixture();

        // customize fixture here

        return fixture;
    }
}

[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public class CustomInlineAutoDataAttribute : InlineAutoDataAttribute
{
    public CustomInlineAutoDataAttribute(params object[] arguments) : base(CreateFixture, arguments) { }

    private IFixture CreateFixture()
    {
        var fixture = new Fixture();

        // customize fixture here

        return fixture;
    }
}
```

With these attribute in place, it is possible to create lean tests without giving up on the customization capabilities of AutoFixture by simply using the new attributes instead of `AutoData` and `InlineAutoData`.

### Dealing with multiple customizations

A problem with this approach is that unit tests requiring different customizations will not be able to use the same custom attribute.
In this case, developers must choose between two options:
- Not using the `AutoData`-approach
- Create a specialization of the attribute for each customization

A bit more advanced alternative is to create a customization method for each unit test and use reflection to invoke that method.

[This gist](https://gist.github.com/Kralizek/cfbe28deacdb5c1e57ad1382e9543805) contains a prototype of said approach. Below is presented a test fixture leveraging it.

```csharp
[TestFixture]
public class Tests
{
    [Test, SmartAutoData(typeof(Tests), nameof(Test1Configuration))]
    public void Test1(string test)
    {
        Assert.That(test, Is.EqualTo("Hello World"));
    }

    static void Test1Configuration(IFixture fixture)
    {
        fixture.Register(() => "Hello World");
    }

    [Test, SmartAutoData]
    public void Test2(string test)
    {
        Assert.That(test, Is.Not.EqualTo("Hello World"));
        Assert.That(test, Is.Not.EqualTo("Foo Bar"));
    }

    [Test, SmartAutoData(typeof(Tests), nameof(Test3Configuration))]
    public void Test3(string test)
    {
        Assert.That(test, Is.EqualTo("Foo Bar"));
    }

    static void Test3Configuration(IFixture fixture)
    {
        fixture.Register(() => "Foo Bar");
    }
}
```

### Order of the parameters
When using the `AutoData` and `InlineAutoData` attributes, the order of parameters of the unit tests assume a critical importance, especially when using the `Frozen` attribute.

For this reason, it's suggested to sort the parameters following this order
- explicit parameters for the `InlineAutoData` attribute (if any)
- frozen parameters for the system under test,
- the system under test
- parameters to be used in the _Act_ phase.

Here is an example

```csharp
public class StringListWrapper 
{
    private readonly IList<string> _list;

    public ListWrapper (IList<string> list) => _list = list;

    public void AddItem(string item, int times)
    {
        for (int i = 0; i < times; i++)
        {
            _list.Add(times);
        }
    }
}

[Test]
[InlineAutoData(1)]
[InlineAutoData(10)]
[InlineAutoData]
public void AddItem_adds_same_item_multiple_times(
    int times, 
    [Frozen] IList<string> innerList, 
    StringListWrapper sut, 
    string item
)
{
    // ACT
    sut.AddItem(item, times);

    // ASSERT
    Assert.That(innerList, Has.AtLeast(times).EqualTo(item));
}
```

## Compatibility issues
Unfortunately, the `AutoData` and `InlineAutoData` attributes are not compatible with the [attributes built into NUnit](Parameterized-tests) shown earlier.