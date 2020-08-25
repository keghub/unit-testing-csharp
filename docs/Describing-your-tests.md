To facilitate working with test suites composed by several tests, NUnit offers different attributes to further describe your fixtures and their tests.

[Here is a list of all the attributes supported in NUnit](https://docs.nunit.org/articles/nunit/writing-tests/attributes.html).

## Category
The `Category` attribute provides an additional dimension to group tests. The attribute can be applied to both a fixture or individual tests. The same test can belong to more than one category.

Most runners offer the possibility to specify which categories to include or exclude in a test run. Tests belonging to excluded categories will not be reported at all.
```csharp
[Test]
[Category("FirstCategory")]
[Category("SecondCategory")]
public void This_is_a_test() { }
```

Optionally, you can create your own category attributes by inheriting from the built-in one. This is useful to avoid the risk of mispell of the category names.

To facilitate this scenario, the `Category` attribute exposes a parameterless protected constructor that will use the attribute name as name of the category.

```csharp
public class ThirdCategoryAttribute : CategoryAttribute { }

public class FourthCategoryAttribute : CategoryAttribute 
{
	public FourthCategoryAttribute() : base("CustomNameForMyFourthCategory") { }	
}
```
You can then proceed using the new attribute as usual

```csharp
[Test]
[ThirdCategory]
[FourthCategory]
public void This_is_another_test() { }
```

[Here is the official documentation for the Category attribute](https://docs.nunit.org/articles/nunit/writing-tests/attributes/category.html).

## Property
Similarly to the `Category` attribute, the `Property` one gives the developer to specify additional properties of the unit test. The attribute can be applied to both a fixture or individual tests.

```csharp
[Test]
[Property("APIVersion", "1.0")]
public void This_is_a_test() { }
```

Developers can also create their own attributes by inheriting the Property attribute and providing the value via the protected constructor while the property name will be inferred from the attribute class name.

```csharp
public class APIVersionAttribute : PropertyAttribute
{
	public APIVersionAttribute(string value) : base(value) { }
}
```
You can then proceed using the new attribute as usual

```csharp
[Test]
[APIVersion("1.0")]
public void This_is_another_test() { }
```

[Here is the official documentation for the Property attribute](https://docs.nunit.org/articles/nunit/writing-tests/attributes/property.html).

## Description
The `Description` attribute lets the developer apply a descriptive test to the test or the whole fixture.
```csharp
[Test]
[Description("This is a human readable description about this test")]
public void This_is_a_test_with_a_description() { }
```

[Here is the official documentation for the Description attribute](https://docs.nunit.org/articles/nunit/writing-tests/attributes/description.html).

## TestOf
The `TestOf` attribute lets the developer specify the class that is being tested. The attribute can be applied to both a fixture or individual tests.
```csharp
[TestFixture]
[TestOf(typeof(PersonStore))]
public class PersonStoreTests { }
```
This attribute can be used by certain IDE to bind tests to tested components.

[Here is the official documentation for the TestOf attribute](https://docs.nunit.org/articles/nunit/writing-tests/attributes/testof.html).