Other than methods, interfaces and non-sealed classes can have properties whose behavior need to be configured. Typically, this is the case of properties part of an interface or abstract or virtual properties of non-sealed classes.

Like for methods, properties can be configured so that unit tests can behave reliably without the need of writing own fakes.

The most basic configuration is the following
```csharp
var mock = new Mock<MyAbstractClass>();
mock.Setup(p => p.Text).Returns("Bar");
mock.Setup(p => p.Value).Returns(123);
```
This will instruct Moq to configure the mocked object so its `Text` property returns the string "Bar".

## Getter and setter methods
Properties are simply artifacts of the C# language to hide behind sugar syntax a getter and a setter method. 

For all intent and purposes, writing
```csharp
public abstract string Name { get; set; }
```
Is equivalent to writing
```csharp
public abstract void SetName(string value);	
public abstract string GetName();
```
(The C# compiler creates methods whose name should not conflict with properly named methods created by developers).

Moq can configure the getter and the setter method independently giving developers very fine grained control of the behavior of the mocked property.
```csharp
mock.SetupGet(p => p.Text) ... ;
mock.SetupSet(p => p.Text = It.IsAny<string>()) ... ;
```
Especially when configuring the setter method, Moq supports the same argument matching capabilities as shown for configuring methods.

To be noted that using Setup on a property is equivalent to `SetupGet`.

## Backing fields and automatic properties
Most properties' getter and setter methods are used to encapsulate a backing field. Typical code (until C# 3.0) would look like the following:
```csharp
private string _text;
public string Text
{
    set { _text = value; }
    get { return _text; }
}
```
C# 3.0 introduced the concept of automatic properties, allowing a much more condensed syntax to express the same behavior.
```csharp
public string Text { get; set; }
```
Like for the getter and setter methods, the C# compiler will generate a backing field whose name will not conflict with other members of the class.

In case a property of the mock should behave like an automatic property, developers can instruct Moq to track the values passed to properties and return them via their getter.
```csharp
mock.SetupProperty(p => p.Text);
var obj = mock.Object;
obj.Text = "Hello world";
Assert.That(obj.Text, Is.EqualTo("Hello world"));
```
Alternatively, developers can also configure a default value for the specified property.
```csharp
mock.SetupProperty(p => p.Text, "Foo bar");
```
Finally, if a mock contains many properties to be configured as automatic properties, developers can use the `SetupAllProperties` to automatically configure all properties of the mock.
```csharp
mock.SetupAllProperties();
```

## Property chains
Some systems use chain of properties, each exposing a mockable type (interface or non-sealed class). You can think of ASP.NET classes like `HttpContextBase`, `HttpResponseBase`, `HttpRequestBase` and so on.

To avoid developers the need for setting up each individual type, Moq supports automatic configuration of mocks.
```csharp
var mock = new Mock<HttpContextBase>();
mock.SetupGet(p => p.Response.Request.UserAgent).Returns("My Browser");
```
In cases like the one above, Moq will take care of creating the needed mocks and configure the properties to return them, leaving the code less cluttered and easier to read.