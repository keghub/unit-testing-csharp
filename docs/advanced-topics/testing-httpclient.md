Due to its peculiar architecture, testing components depending on `HttpClient` is quite complicated.

To understand the challenges, it is important to keep in mind that `HttpClient` is not a service that can be mocked (in fact, it does not implement any interface), rather a facade around a pipeline of instances of `HttpMessageHandler` subclasses. [Here is a series of posts by Steve Gordon](https://www.stevejgordon.co.uk/tag/httpclient) that touch the complexity of the `HttpClient` and related components.

Mocking `HttpClient` in reality equates to creating an instance of `HttpClient` with a fake `HttpMessageHandler`. Unfortunately, this is easier said than done as there are many aspects to be taken into consideration when creating a fake `HttpMessageHandler`.

The library [MockHttp](https://github.com/richardszalay/mockhttp) exposes a subclass of the `HttpMessageHandler` abstract class specifically designed to facilitate unit testing, providing a fluent configuration API. Since the system under test is consuming the `HttpClient`, it will remain unaware that the `MockHttpMessageHandler` is being used.

Let's consider this service using `HttpClient` as test subject

```csharp
public class Service
{
    private readonly HttpClient _http;
    
    public Service(HttpClient http)
    {
        _http = http ?? throw new ArgumentNullException(nameof(http));
    }
    
    public Task<string> GetStringAsync(Uri uri)
    {
        return _http.GetStringAsync(uri);
    }
}
```

We can write a test that asserts that `GetStringAsync` retrieves the content from a certain URI and returns it as string. By using `MockHttp`, we can arrange the handler so that when a `GET` request is issued to the given URI, a given content is returned.

```csharp
[Test]
public async Task GetStringAsync_uses_HttpClient_to_get_content_from_given_URI()
{
    // ARRANGE
    var fixture = new Fixture();
    var testUri = fixture.Create<Uri>();
    var expectedResult = fixture.Create<string>();

    var handler = new MockHttpMessageHandler();
    handler.When(HttpMethod.Get, testUri.ToString())
           .Respond(HttpStatusCode.OK, new StringContent(expectedResult));

    var http = handler.ToHttpClient();

    var sut = new Service(http);

    // ACT
    var result = await sut.GetStringAsync(testUri);

    // ASSERT
    Assert.That(result, Is.EqualTo(expectedResult));
}
```

The same test can be written in a much more concise form by leveraging a [glue-library for AutoFixture that uses MockHttp](https://github.com/Kralizek/AutoFixtureExtensions/tree/master/src/MockHttp). This library can be obtained from NuGet under the name of [`Kralizek.AutoFixture.Extensions.MockHttp`](https://www.nuget.org/packages/Kralizek.AutoFixture.Extensions.MockHttp/). The library is built around a specimen builder that, when asked for an `HttpClient`, creates one that uses a `MockHttpMessageHandler` internally. Since the `MockHttpMessageHandler` is resolved using AutoFixture, we can leverage patterns like freezing and injection to retain access to the internal instance.

```csharp
[Test]
public async Task GetStringAsync_uses_HttpClient_to_get_content_from_given_URI()
{
    // ARRANGE
    var fixture = new Fixture().AddMockHttp();

    var testUri = fixture.Create<Uri>();
    var expectedResult = fixture.Create<string>();

    var handler = fixture.Freeze<MockHttpMessageHandler>();
    handler.When(HttpMethod.Get, testUri.ToString())
           .Respond(HttpStatusCode.OK, new StringContent(expectedResult));

    var sut = fixture.Create<Service>();

    // ACT
    var result = await sut.GetStringAsync(testUri);

    // ASSERT
    Assert.That(result, Is.EqualTo(expectedResult));
}
```

Things get really interesting when we combine MockHttp with a `AutoData` attribute

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class HttpAutoDataAttribute : AutoDataAttribute
{
    public HttpAutoDataAttribute() : base (CreateFixture) {}

    private IFixture CreateFixture()
    {
        var fixture = new Fixture();

        fixture.AddMockHttp();

        return fixture;
    }
}
```

With the attribute defined, we can rewrite the unit test like this:

```csharp
[Test, HttpAutoData]
public async Task GetStringAsync_uses_HttpClient_to_get_content_from_given_URI([Frozen] MockHttpMessageHandler handler, Service sut, Uri testUri, string expectedResult)
{
    // ARRANGE
    handler.When(HttpMethod.Get, testUri.ToString())
           .Respond(HttpStatusCode.OK, new StringContent(expectedResult));

    // ACT
    var result = await sut.GetStringAsync(testUri);

    // ASSERT
    Assert.That(result, Is.EqualTo(expectedResult));
}
```

As shown in the snippet above, by leveraging the integration between AutoFixture, NUnit and MockHttp, it is possible to write very concise yet powerful tests for components using the `HttpClient` class and its related components.

## HttpClientFactory

To face some of the issues caused by bad usage of `HttpClient` (like socket exhaustion and DNS cache pinning), Microsoft included in .NET 2.1 a new API often referred to as `HttpClientFactory`.

Developers can leverage the new API in two ways:
- by instructing the dependency injection engine to use the `HttpClientFactory` when requested an instance of `HttpClient`
- by replacing the dependency to be a `IHttpClientFactory` and use it to fetch an instance of `HttpClient`

Since the first approach deosn't alter the test subject, the same setup as shown above can be used.

On the other hand, consuming an `IHttpClientFactory` requires some changes to the unit tests.

Here is the test subject modified so that it consumes an `IHttpClientFactory`.

```csharp
public class Service
{
    private readonly IHttpClientFactory _httpFactory;
    
    public Service(IHttpClientFactory httpFactory)
    {
        _httpFactory = httpFactory ?? throw new ArgumentNullException(nameof(httpFactory));
    }
    
    public Task<string> GetStringAsync(Uri uri)
    {
        var http = _httpFactory.CreateClient(nameof(Service));
        
        return http.GetStringAsync(uri);
    }
}
```

Since we're dealing with an interface, we can use Moq to build a test fake.

```csharp
public async Task GetStringAsync_uses_HttpClient_to_get_content_from_given_URI()
{
    // ARRANGE
    var fixture = new Fixture();
    var testUri = fixture.Create<Uri>();
    var expectedResult = fixture.Create<string>();

    var handler = new MockHttpMessageHandler();
    handler.When(HttpMethod.Get, testUri.ToString())
           .Respond(HttpStatusCode.OK, new StringContent(expectedResult));
    
    var http = handler.ToHttpClient();
    
    var mockHttpClientFactory = new Mock<IHttpClientFactory>();
    mockHttpClientFactory.Setup(p => p.CreateClient(It.IsAny<string>())).Returns(http);
    
    var sut = new Service(mockHttpClientFactory.Object);
    
    // ACT
    var result = await sut.GetStringAsync(testUri);
    
    // ASSERT
    Assert.That(result, Is.EqualTo(expectedResult));
}
```

Quite interestingly, when converting the unit test above into one leveraging the `AutoData` attribute, we get the exact same unit test as the complexity of `IHttpClientFactory.CreateClient` is handled automatically by Moq, once AutoMoq is configured accordingly.

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class HttpAutoDataAttribute : AutoDataAttribute
{
    public HttpAutoDataAttribute() : base (CreateFixture) {}

    private IFixture CreateFixture()
    {
        var fixture = new Fixture();

        fixture.AddMockHttp();

        fixture.Customize(new AutoMoqCustomization { ConfigureMembers = true, GenerateDelegates = true });

        return fixture;
    }
}

[Test, HttpAutoData]
public async Task GetStringAsync_uses_HttpClient_to_get_content_from_given_URI([Frozen] MockHttpMessageHandler handler, Service sut, Uri testUri, string expectedResult)
{
    // ARRANGE
    handler.When(HttpMethod.Get, testUri.ToString())
           .Respond(HttpStatusCode.OK, new StringContent(expectedResult));

    // ACT
    var result = await sut.GetStringAsync(testUri);

    // ASSERT
    Assert.That(result, Is.EqualTo(expectedResult));
}
```