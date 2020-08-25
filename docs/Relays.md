AutoFixture is focused on creating instances of concrete types. For this reason it doesn't have any support for non-concrete types like interfaces and abstract classes.

Most of the time, AutoFixture needs to resort to other libraries, like Moq, to create objects implementing an interface or representing a dummy specialization of an abstract class.

Sometimes, it's sufficient to return a concrete type whenever a non-concrete type is requested. To handle cases like this, AutoFixture offers the class `TypeRelay`.

A `TypeRelay` is a customization that can be fed to the Fixture that simply maps relays requests of a certain type to another one.

```csharp
public abstract class Animal { }

public class Dog : Animal { }

[Test]
public void Fixture_should_return_relayed_type()
{
    var fixture = new Fixture();

    fixture.Customizations.Add(new TypeRelay(typeof(Animal), typeof(Dog)));

    var animal = fixture.Create<Animal>();

    Assert.That(animal, Is.InstanceOf<Dog>());
}
```

AutoFixture uses relays to automatically register well-known BCL interfaces to concrete types. The most common use cases are [collection interfaces](Default-configurations#collections).