# Data-annotations

Data annotations were introduced with ASP.NET MVC as a declarative way to describe validation rules.

By using data annotation attributes, developers can decorate their models with validation rules that ASP.NET would use to generate validation logic both on the client and on the server.

```csharp
public class Contact
{
    [StringLength(10)]
    public string FirstName { get; set; }

    [StringLength(10)]
    public string LastName { get; set; }

    [Range(18, 30)]
    public int Age { get; set; }

    [RegularExpression(@"^[2-9]\d{2}-\d{3}-\d{4}$")]
    public string Telephone { get; set; }
}
```

AutoFixture has a built-in support for some of the data annotation attributes:

* `StringLength` sets the maximum length of a string. AutoFixture will generate a string whose length is less or equal than the specified value.
* `Range` sets the boundaries of a valid range of values. AutoFixture will generate a value between the specified boundaries.
* `RegularExpression` specifies that a string must match a certain pattern. AutoFixture will generate a string that matches the specified pattern.

Since the support is built-in, types decorated with the supported data annotation attributes are automatically handled.

```csharp
var fixture = new Fixture();
var contact = fixture.Create<Contact>();
```

The snippet above will generate an object whose properties will match the specified limits.

