# Default configurations

AutoFixture comes with a set of default builders so that most common scenarios are supported out-of-the-box.

These builders can be overridden with ad-hoc customizations or when building an anonymous variable.

## Numbers

The default builder for numbers is the [`RandomNumericSequenceGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/RandomNumericSequenceGenerator.cs).

It supports the following types: `Byte`, `Decimal`, `Double`, `Int16`, `Int32`, `Int64`, `SByte`, `Single`, `UInt16`, `UInt32`, `UInt64`.

Unique numbers are generated randomly from the set `[1, 255]`. Once these are used up they are then be generated from the set `[256, 65 535]`. And finally from the set `[65 536, 2 147 483 647]`.

When all numbers within the final set have been used AutoFixture will start again from the first set.

## Chars

The default builder for chars is the [`RandomCharSequenceGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/RandomCharSequenceGenerator.cs).

It generates random characters from the printable ASCII character set \(`!` \(33\) to `~` \(126\)\).

## Strings

The default builder for strings is an instance of [`StringGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/StringGenerator.cs).

It returns randomly generated `Guid` as string using the format `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

## Dates and times

The default builder for `DateTime` is the [`RandomDateTimeSequenceGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/RandomDateTimeSequenceGenerator.cs).

It generates a date and a time between 2 years prior and 2 years after the current date.

```csharp
var dateTime = fixture.Create<DateTime>();
Assert.That(dateTime, Is.GreaterOrEqualThan(DateTime.Today.AddYears(-2)));
Assert.That(dateTime, Is.LessOrEqualThan(DateTime.Today.AddYears(2)));
```

## Enums

The [`EnumGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/EnumGenerator.cs) is the default builder for enumerations.

It returns all the values of the enumeration in order. When all values are used, it starts from the first value again.

## Collections

AutoFixture has a built-in support for many collection types: `Dictionary`, `SortedDictionary`, `SortedList`, `Collection`, `List`, `HashSet`, `SortedSet`, `ObservableCollection`, and `Array`.

Also, AutoFixture natively supports common collection interfaces like `IDictionary`, `IReadOnlyDictionary`, `ICollection`, `IReadOnlyCollection`, `IList`, `IReadOnlyList`, `ISet`, and `IEnumerable`. When requested, AutoFixture will return a type implementing the requested interface.

## Tuple and ValueTuple

AutoFixture is able to generate out-of-the-box anonymous variables of both `Tuple` and `ValueTuple` types.

```csharp
var tuple = fixture.Create<Tuple<string, int>>();

var valueTuple = fixture.Create<ValueTuple<string, int>>();
```

In the case of `ValueTuple`, it also supports the simplified syntax.

```csharp
var valueTuple = fixture.Create<(string, int)>();
```

## Uri

The default builder for `Uri` is the [`UriGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/UriGenerator.cs).

The `UriGenerator` delegates the scheme creation to a `UriSchemeGenerator` configured to return `http` by default.

## MailAddress

The default builder for the `MailAddress` type is the [`MailAddressGenerator`](https://github.com/AutoFixture/AutoFixture/blob/master/Src/AutoFixture/MailAddressGenerator.cs).

This generator splits the email address into two parts that can be independently customized:

* a `EmailAddressLocalPart`, generated via a `EmailAddressLocalPartGenerator`
* a `DomainName`, generated via a `DomainNameGenerator`

## Fixed values

For some types, the default configuration of AutoFixture is to return well-known values.

Here is a list of these cases:

* `IPAddress` is registered with the value `IPAddress.Loopback`
* `Encoding` is registered with the value `Encoding.UTF8`
* `CultureInfo` is registered with the value `CultureInfo.InvariantCulture`

