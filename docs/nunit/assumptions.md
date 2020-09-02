# Assumptions

Sometimes, unit tests can be relevant only when certain conditions are met. This is often the case with [parameterized tests](./parameterized-tests.md).

In these cases, mapping these conditions as assertions would mark as failed tests that simply received invalid data. To obviate this issue, NUnit offers the possibility to make _assumptions_ on the incoming data. Unlike assertions, unmet assumptions make the runner mark the test as _Invalid_ instead of _Failed_.

```csharp
[Test]
public void A_test_with_assumptions (
    [Values(2020, 2025, 2030)] int year,
    [Range(1, 12)] int month,
    [Range(1, 31)] int day)
{
    Assume.That(day, Is.LessOrEqualThan(DateTime.DaysInMonth(year, month)));

    ...
}
```

In the example above, test runs with invalid day/month/year combinations \(like June 31st\) are ignored.

Please note that assumptions make use of the `Assume` static class. `Assume.That` has the same set of overloads as `Assert.That`.

