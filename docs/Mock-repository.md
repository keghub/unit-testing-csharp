Working with several mocks can be very tedious. Especially when each needs to be configured (behavior and default value provider) and verified.

To facilitate this aspect, developers can use the MockRepository to create, customize and verify mocks as needed.
```csharp
var repository = new MockRepository(MockBehavior.Strict) { DefaultValue = DefaultValue.Mock };
```
A repository can be used to create new mocks (and override the default setting if needed)
```csharp
var mockFoo = repository.Create<IFoo>();
var mockBar = repository.Create<IBar>(MockBehavior.Loose);
```
Also, a repository can be used to verify the expectations configured on all mocks created by it
```csharp
repository.Verify();
```