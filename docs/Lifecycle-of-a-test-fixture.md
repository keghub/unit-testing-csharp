As mentioned before, NUnit gives the developer the possibility to extract all initialization and tear-down code that multiple tests might be sharing into ad-hoc methods.

Developers can take advantage of the following facilities to streamline their fixtures
- A method decorated with a `SetUp` attribute will be executed before each test
- A method decorated with a `TearDown` attribute will be executed after each test
- A method decorated with a `OneTimeSetUp` attribute will be executed before any test is executed
- A method decorated with a `OneTimeTearDown` attribute will be executed after all tests have been executed 
- The class constructor will be executed before any method and can be used to prepare fields that shouldn't be modified while executing the tests.

Additionally, developers can set up fixtures contained in a namespace and all its children by creating a class decorated with the attribute `SetUpFixture`. This class will be able to contain methods decorated with `OneTimeSetUp` and `OneTimeTearDown` attributes.

NUnit supports multiple `SetUpFixture` classes: in this case, setup methods will be executed starting from the most external namespace in and the teardown from the most internal namespace out.

## Example
Let's execute all tests contained in the snippet below:
```csharp
using System.Diagnostics;
using NUnit.Framework;

[SetUpFixture]
public class RootFixtureSetup
{
    [OneTimeSetUp]
    public void OneTimeSetUp() => Debug.WriteLine("RootFixtureSetup:OneTimeSetUp");

    [OneTimeTearDown]
    public void OneTimeTearDown() => Debug.WriteLine("RootFixtureSetup:OneTimeTearDown");
}

namespace TestLifeCycle
{
    [SetUpFixture]
    public class FixtureSetup
    {
        [OneTimeSetUp]
        public void OneTimeSetUp() => Debug.WriteLine("FixtureSetup:OneTimeSetUp");

        [OneTimeTearDown]
        public void OneTimeTearDown() => Debug.WriteLine("FixtureSetup:OneTimeTearDown");
    }

    [TestFixture]
    public class Tests
    {
        [OneTimeSetUp]
        public void OneTimeSetUp() => Debug.WriteLine("Tests:OneTimeSetUp");

        [SetUp]
        public void Setup() => Debug.WriteLine("Tests:SetUp");

        public Tests() => Debug.WriteLine("Tests:Constructor");

        [Test]
        public void Test1() => Debug.WriteLine("Tests:Test1");

        [Test]
        public void Test2() => Debug.WriteLine("Tests:Test2");

        [TearDown]
        public void TearDown() => Debug.WriteLine("Tests:TearDown");

        [OneTimeTearDown]
        public void OneTimeTearDown() => Debug.WriteLine("Tests:OneTimeTearDown");
    }
}
```

We will see the following output on the console.
```
RootFixtureSetup:OneTimeSetUp
FixtureSetup:OneTimeSetUp
Tests:Constructor
Tests:OneTimeSetUp
Tests:SetUp
Tests:Test1
Tests:TearDown
Tests:SetUp
Tests:Test2
Tests:TearDown
Tests:OneTimeTearDown
FixtureSetup:OneTimeTearDown
RootFixtureSetup:OneTimeTearDown
```