AutoFixture gives its users the ability to quickly create anonymous variables or to customize how they are created, totally or partially.

The snippets below will be based on the following custom types
```csharp
public class Person
{
    public string FirstName { get; set; }

    public string MiddleName { get; set; }

    public string LastName { get; set; }

    public DateTime DateOfBirth { get; set; }

    public Pet[] Pets { get; set; } = Array.Empty<Pet>();
}

public class Pet { ... }

public class Car
{
    public void AssignOwner(Person person) => Owner = owner;

    public Person Owner { get; }
}
```

## `Create`

The `Create` method is responsible for initiating the construction of the requested type. When invoked, AutoFixture will use all current customizations and default configurations.

```csharp
var message = fixture.Create<string>();

Assert.That(message, Is.Not.Empty);
```

AutoFixture supports also complex types, even when not part of the BCL.

```csharp
var person = fixture.Create<Person>();

Assert.That(person.FirstName, Is.Not.Null);
Assert.That(person.LastName, Is.Not.Null);
```

## `Build`

Before creating the object, the Build method can be used to add one-time customizations to be used for the creation of the next variable. Once the object is created, the customizations are lost

```csharp
var customizedPerson = fixture.Build<Person>()
                              .With(p => p.FirstName, "John")
                              .Create();

var anotherPerson = fixture.Create<Person>();

Assert.That(customizedPerson.FirstName, Is.EqualTo("John"));
Assert.That(anotherPerson.FirstName, Is.Not.EqualTo("John"));
```

As shown in the snippet, `Build` is used to initiate the fluent Customization API. This API is composed by many methods.

### `OmitAutoProperties`

`OmitAutoProperties` disables the assignment of values to properties.

```csharp
var person = fixture.Build<Person>()
                    .OmitAutoProperties()
                    .Create();

Assert.That(person.FirstName, Is.Null);
Assert.That(person.LastName, Is.Null);
```

### `WithAutoProperties`
Opposite to `OmitAutoProperties`, `WithAutoProperties` forces the assignment of values to properties, even if it had been disabled on the fixture

```csharp
fixture.OmitAutoProperties = true;

var person = fixture.Build<Person>()
                    .WithAutoProperties()
                    .Create();

Assert.That(person.FirstName, Is.Not.Null);
```

### `With` 

The `With` construct allows the customization of writeable properties and public fields.

There are different overloads, each with its own use case and semantic.

#### No value specified
This overload is used to signal AutoFixture that the selected property should receive a value even if property generation was disabled earlier for the whole type with `OmitAutoProperties()` or by setting the `OmitAutoProperties` property of the fixture to `false`.
```csharp
var person = fixture.Build<Person>()
                    .OmitAutoProperties()
                    .With(p => p.FirstName)
                    .Create();

Assert.That(person.FirstName, Is.Not.Null);
Assert.That(person.LastName, Is.Null);
```

#### With a specified value
This overload is used to assign a specific value to the selected property.
```csharp
var person = fixture.Build<Person>()
                    .With(p => p.FirstName, "John")
                    .Create();

Assert.That(person.FirstName, Is.EqualTo("John"));
```

#### With a factory method
This overload is used to provide a lazily-evaluated factory method used to generate the value of the property. The factory method can have either no or one parameter. If requested, AutoFixture will create and provide an instance of the requested argument type.
```csharp
var person = fixture.Build<Person>()
                    .With(p => p.FirstName, (int ordinal) => $"John {ordinal}")
                    .Create();
```

If more than one argument is needed, developers can request for a tuple composed by the needed types.
```csharp
var person = fixture.Build<Person>()
                    .With(p => p.FirstName, ((string seed, int ordinal) p) => $"{p.seed} {p.ordinal}")
                    .Create();
```

Finally, developers can request for an instance of `IFixture`. If so, the same instance will be passed.
```csharp
var person = fixture.Build<Person>()
                    .With(p => p.FirstName, (IFixture f) => $"John {f.Create<int>()}")
                    .Create();
```

### `Without`

The `Without` construct can be used to disable the generation of the value for a specific property without affecting other properties.

```csharp
var person = fixture.Build<Person>()
                    .Without(p => p.MiddleName)
                    .Create();

Assert.That(person.MiddleName, Is.Null);
``` 

Please notice the difference between `Without` and `With(null)`: the first skips the selected properties (leaving the default value) whilst the second assigns it a `null`.

```csharp
var person1 = fixture.Build<Person>()
                     .Without(p => p.Pets)
                     .Create();

Assert.That(person1.Pets, Is.Not.Null.And.Empty);

var person2 = fixture.Build<Person>()
                     .With(p => p.Pets, null as Pet[])
                     .Create();

Assert.That(person2.Pets, Is.Null);
```

Also, since delegates can be null too, developers need to cast null to the type of the property to help the compiler picking the right overload.

### `FromFactory`

Some types are too complex to build for AutoFixture to guess. In these cases developers can use the `FromFactory` construct to instruct AutoFixture.

Like for the `With` construct, `FromFactory` allows the developer to specify a function delegate that will be used to generate the object. There are overloads who accept from 0 to 4 arguments. AutoFixture will provide a value for each requested argument.

```csharp
var person = fixture.Build<Person>()
                    .FromFactory((string firstName, string lastName) => new Person { FirstName = firstName, LastName = lastName })
                    .Create();
```

On top of these overloads, `FromFactory` has an overload accepting an `ISpecimenBuilder` for more advanced scenarios.

### `Do`

If the value to be built requires some methods to be invoked as part of its creation, you can use the `Do` construct to instruct AutoFixture to do so.

```csharp
var person = fixture.Create<Person>();

var car = fixture.Build<Car>()
                 .Do(c => c.AssignOwner(person))
                 .Create();

Assert.That(car.Owner, Is.SameAs(person));
```

[You can find additional information about `Do` on this blog post from the author of AutoFixture](https://blog.ploeh.dk/2009/06/09/CallingMethodsWhileBuildingAnonymousVariablesWithAutoFixture/)