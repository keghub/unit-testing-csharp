AutoFixture is focused on creating instances of concrete types. For this reason it doesn't have any support for non-concrete types like interfaces and abstract classes.

By default, AutoFixture is only able to work with concrete types. In some cases, interfaces and abstract classes have an implementation or a subtype that can be used as default. To handle cases like this, AutoFixture offers the class `TypeRelay`.

A `TypeRelay` is a customization that can be used to instruct the Fixture to relay requests of a certain type to another one.

```csharp
public abstract class Animal { }

public class Dog : Animal { }

[Test]
public void Fixture_should_return_relayed_type()
{
    // ARRANGE
    var fixture = new Fixture();
    fixture.Customizations.Add(new TypeRelay(typeof(Animal), typeof(Dog)));

    // ACT
    var animal = fixture.Create<Animal>();

    // ASSERT
    Assert.That(animal, Is.InstanceOf<Dog>());
}
```

AutoFixture uses relays to support well-known interfaces like `IList<T>`. The [collection interfaces](Default-configurations#collections) are forwarded to their most common implementation.