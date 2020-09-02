# Parameterized tests

NUnit supports parameterized tests: tests who accepts parameters. These tests are convenient because they give the possibility to execute the same test against different set of parameters.

A typical example is validating email addresses: by specifying multiple inputs, you can ensure the validation logic is tested against all corner cases without the need of rewriting the full unit test.

NUnit provides multiple attributes to specify which values should be supplied to each parameter. When exploring the test project, the attributes will be combined to determine the concrete tests to be executed.

These attributes can be divided in two major categories: those who provide complete test cases and those who provide data to a single parameter.

## `TestCase` attribute

The `TestCase` attribute allows the developer to specify the values to be passed to the unit test by simply embedding them in the attribute signature.

Due to the constraints of attributes in C\#, only selected types can be specified in an attribute \([see documentation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/attributes#attribute-parameter-types)\). This method is suggested when your test parameters are simple values.

In the sample below, NUnit will find three unit tests, one for each test email address provided by the `TestCase` attribute.

```csharp
[Test]
[TestCase("hello@welcome.com")]
[TestCase("hello+plus@welcome.com")]
[TestCase("hello.with.dots@welcome.com")]
public void ValidateEmail_supports_common_email_addresses(string emailAddress)
{
    // ARRANGE
    var sut = new EmailValidator();

    // ACT
    var result = sut.ValidateEmail(emailAddress);

    // ASSERT
    Assert.That(result.IsValid, Is.True);
}
```

You can check the official documentation for additional information \([see documentation](https://github.com/nunit/docs/wiki/TestCase-Attribute)\).

## `TestCaseSource` attribute

The `TestCaseSource` attribute also provides complete test cases but unlike the `TestCase` attribute, it points at an external method. Not being limited by language constraints, complex types can be provided.

The attribute supports three patterns:

* Use a public static method in the same class of the test by specifying its name
* Use a public static method in another class by specifying the class type and the method name
* Use an external class implementing the `IEnumerable` interface

Here is an example of the first case.

```csharp
[Test, TestCaseSource("GetEmailAddresses")]
public void ValidateEmail_supports_common_email_addresses(MailAddress emailAddress)
{
    // ARRANGE
    var sut = new EmailValidator();

    // ACT
    var result = sut.ValidateEmail(emailAddress);

    // ASSERT
    Assert.That(result.IsValid, Is.True);
}

public static IEnumerable<MailAddress> GetEmailAddresses()
{
    yield return new MailAddress("hello@welcome.com");
    yield return new MailAddress("hello+plus@welcome.com");
    yield return new MailAddress("hello.with.dots@welcome.com");
}
```

More information can be found on the official documentation page for the `TestCaseSource` attribute \([see documentation](https://github.com/nunit/docs/wiki/TestCaseSource-Attribute)\).

## Inline value attributes

Sometimes the [cardinality](https://en.wikipedia.org/wiki/Cardinality) of the parameters of a unit test is such that creating a test case for each combination of valid values would be a tedious job.

Luckily, NUnit comes with a set of parameter attributes that tell the test runner to generate a test for each value. If more than one parameter is decorated with these attributes, the test runner will generate a unit test execution for each combination of the parameter values.

By default, NUnit includes three attributes that support inlined values.

* The [`Random`](https://github.com/nunit/docs/wiki/Random-Attribute) attribute will generate the amount of random values specified by the constructor. Optionally, the values can be constrained within a given range. This attribute supports numeric types only.
* The [`Range`](https://github.com/nunit/docs/wiki/Range-Attribute) attribute will generate all values between the two given boundaries of the range. This attribute supports numeric types only.
* The [`Values`](https://github.com/nunit/docs/wiki/Values-Attribute) attribute will return all the specified values. For _Boolean_ and _Enum_ parameters, the attribute will generate all possible values unless some of them are specified. Since the values are inlined, the same restrictions of the `TestCase` attribute apply.

The example below will instruct the NUnit runner to generate 252 test by combining the cardinality of all given parameters.

```csharp
[Test]
public void Test_with_many_parameters (

    // this will generate 2 values: true, false
    [Values] bool flag,

    // this will generate 3 values: Monday, Wednesday, Friday
    [Values(DayOfWeek.Monday, DayOfWeek.Wednesday, DayOfWeek.Friday)] DayOfWeek dayOfWeek,

    // this will generate 2 values: X, Y
    [Values("X", "Y")] string testLetter,

    // this will generate 3 random values between -10 and 10
    [Random(-10, 10, 3)] int randomValue,

    // this will generate 7 values: -3, -2, -1, 0, 1, 2, 3
    [Range(-3, 1, 3)] int rangeValue,
    )
{
    Console.WriteLine($"{flag} {dayOfWeek} {testLetter} {randomValue} {rangeValue}");
    Assert.Pass();
}
```

## `ValueSource` attribute

Unlike the `Values` attribute, the `ValueSource` attribute allows you to specify a generator method that will generate values for the decorated parameter of the test.

Similarly to the `TestCaseSource` attribute, it supports the following patterns:

* Use a public static method in the same class of the test by specifying its name
* Use a public static method in another class by specifying the class type and the method name

The advantage of the `ValueSource` attribute over the `TestCaseSource` attribute is that you can combine with other parameter value attributes. Also, it allows reusing the same generator method over multiple unit tests.

The example below will generate 6 tests \(3 email addresses x 2 days of the week\).

```csharp
[Test]
public void Test_with_many_parameters (
    [ValueSource(nameof(GetEmailAddresses))] MailAddress emailAddress,
    [Values(DayOfWeek.Saturday, DayOfWeek.Sunday)] DayOfWeek dayOfWeek
)
{
    Assert.Pass("{0} {1}", emailAddress, dayOfWeek);
}

public IEnumerable<MailAddress> GetEmailAddresses()
{
    yield return new MailAddress("hello@welcome.com");
    yield return new MailAddress("hello+plus@welcome.com");
    yield return new MailAddress("hello.with.dots@welcome.com");
}
```

More information can be found on the official documentation page for the `ValueSource` attribute \([see documentation](https://github.com/nunit/docs/wiki/ValueSource-Attribute)\).

