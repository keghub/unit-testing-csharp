# Type-customization

The same fluent API used to specify [how to build an anonymous variable](https://github.com/emgdev/unit-testing-csharp/tree/c1e06f02ecb67288bafa6a2fe26e4d233f910b0e/docs/Create-and-Build/README.md#Build) can be used to instruct AutoFixture how to create every instance of the same type.

This can be done using the `Customize<T>` method of `IFixture`.

```csharp
var fixture = new Fixture();

fixture.Customize<Person>(c => c.With(p => p.FirstName, "John"));

var person = fixture.Create<Person>();

Assert.That(person.FirstName, Is.EqualTo("John"));
```

These type-wide customizations can be overridden by customizing the object creation via `Build<T>`.

```csharp
var fixture = new Fixture();

fixture.Customize<Person>(c => c.With(p => p.FirstName, "John")
                                .With(p => p.LastName, "Smith"));

var person = fixture.Build<Person>()
                    .With(p => p.FirstName, "Sam")
                    .Create();

Assert.That(person.FirstName, Is.EqualTo("Sam"));
Assert.That(person.LastName, Is.EqualTo("Smith"));
```

## `Register`

Certain customizations are so common that AutoFixture offers shortcuts to them.

The `Register` extension method can be used as replacement for a customization composed by a call to `FromFactory` followed by `OmitAutoProperties`.

```csharp
fixture.Customize<Person>(c => c
    .FromFactory((string firstName, string lastName) => new Person { FirstName = firstName, LastName = lastName })
    .OmitAutoProperties());

fixture.Register<Person>((string firstName, string lastName) => new Person { FirstName = firstName, LastName = lastName });
```

In the snippet above, the two commands are equivalent.

`Register` can be used to handle custom interfaces since they are not natively supported by AutoFixture.

```csharp
fixture.Register<IService>(() => new FakeService());
```

This approach works pretty well with simple scenarios \(with simple or no dependencies\). To handle more complex scenarios, it's better to use [relays](https://github.com/emgdev/unit-testing-csharp/tree/c1e06f02ecb67288bafa6a2fe26e4d233f910b0e/docs/Relays/README.md).

Finally, the generator method can also return a subtype of the registered type.

```csharp
fixture.Register<Animal>(() => new Dog());
```

## `Inject`

Another common scenario is customizing a type so that the same instance is returned at every request.

The `Inject` extension method can be used as a replacement for a call to `Register` that uses an instance already existing.

```csharp
var person = fixture.Create<Person>();

fixture.Register<Person>(() => person);
```

The snippet above can be replaced with

```csharp
var person = fixture.Create<Person>();
fixture.Inject(person);
```

Like `Register`, `Inject` can accept a subtype of the type being configured.

```csharp
var dog = fixture.Create<Dog>();
fixture.Inject<Animal>(dog);
```

Finally, `Inject` is the best way to force a value to an `Enum`.

## `Freeze`

Given how common the Create/Inject combination is, AutoFixture offers a shortcut.

The `Freeze` method creates an anonymous variable, injects it and returns it to the caller.

```csharp
fixture.Inject(fixture.Create<Person>());

fixture.Freeze<Person>();
```

In the snippet above, the two commands are equivalent.

To be noted that `Freeze` returns the frozen instance. We will see how this will be useful when integrating AutoFixture with NUnit.

```csharp
public T Freeze<T>(this IFixture fixture)
{
   T item = fixture.Create<T>();
   fixture.Inject(item);
   return item;
}
```

