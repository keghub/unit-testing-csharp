# Quick-glance-at-AutoFixture

AutoFixture is a library focused on test data generation so that unit tests can focus on the behavior to be tested rather than on the plumbing needed to get the test to run.

There is an exhaustive cheat sheet available [here](https://github.com/AutoFixture/AutoFixture/wiki/Cheat-Sheet) for quick reference.

Note: at the time of writing, the latest version of AutoFixture is the 4.13.

## Quick sample

This is the simplest usage of AutoFixture

```csharp
[Test]
public void Echo_should_return_incoming_message()
{
    // ARRANGE
    var fixture = new Fixture();
    var sut = fixture.Create<EchoService>();
    var message = fixture.Create<string>();

    // ACT
    var result = sut.Echo(message);

    // ASSERT
    Assert.That(result, Is.EqualTo(message));
}
```

In the snippet above, 1. we create an instance of `Fixture` 2. we use the fixture to create an instance of the system under test, the service EchoService 3. we use the fixture to create a sample message 4. we exert the system under test by passing the sample message 5. we assert that the output is equal to the input

