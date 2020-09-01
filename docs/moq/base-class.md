# Base class

Especially in older codebases, services and components can be part of a deep hierarchy of classes. Often, belonging to a class hierarchy makes it so a class inherits requirements and behaviors from its ancestors.

Each mock offers properties and constructor overloads to accommodate these requirements, such as constructor arguments, and retain the needed behaviors.

For this section, the following class will be used

```csharp
public class Person 
{
    public Person(string name)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
    }

    public string Name { get; }

    public virtual void Greet() => Debug.WriteLine($"Hello, I'm {Name}.");
}
```

Please notice that the method `Greet` is defined as virtual, thus allowing subclasses to redefine its behavior.

## Constructor arguments

Generally, when mocking interfaces, there is no need to specify constructor arguments since the mock is backed by a class created with a parameterless constructor.

Unfortunately, this might not be the case when mocking classes.

In this case, developers can specify the constructor arguments when creating the mock.

```csharp
var mock = new Mock<Person>("John");
```

The snippet above will create a mock backed by a class inheriting from `Person` even if it requires a string in its constructor.

## Preserve virtual methods' behavior

Sometimes the classes to be mocked present one or more virtual methods whose default implementation already satisfies the needs of the mock. In cases like this it makes sense to preserve the original behavior, rather than let Moq override the method.

Moq offers two ways to achieve this.

The first one is by configuring each method accordingly by using the CallBase construct.

```csharp
mock.Setup(p => p.Greet()).CallBase();
```

Alternatively, developers can instruct Moq to always resort to the base class behavior, whenever there is one.

```csharp
var mock = new Mock<Person> { CallBase = true };
```

This approach is particularly useful when members of the base class are not publicly visible.

