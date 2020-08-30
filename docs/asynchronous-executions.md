# Asynchronous-executions

C\# 5.0 baked asynchrony into the language by introducing the keywords `async` and `await`. Given how asynchronous code is handled in C\#, many components now expose asynchronous methods that need to be properly tested.

NUnit supports out of the box asynchronous methods by wrapping properly executing them in an asynchronous context and unwrapping eventual exception raised by the code itself or by failed assertions. To make sure the runner properly waits for the completion of the asynchronous tests, these cannot be `async void` methods.

Here are two tests testing the synchronous and asynchronous methods of the same component.

```csharp
public class FileLoader
{
    public string Load(string filePath) { ... }

    public Task<string> LoadAsync(string filePath) { ... }
}

[Test]
public void Load_retrieves_file()
{
    // ARRANGE
    var fileName = "my_file.txt";

    var sut = new FileLoader();

    // ACT
    var result = sut.Load(fileName);

    // ASSERT        
    Assert.That(result, Is.Not.Null);
}

[Test]
public async Task LoadAsync_retrieves_file()
{
    // ARRANGE
    var fileName = "my_file.txt";

    var sut = new FileLoader();

    // ACT
    var result = await sut.LoadAsync(fileName);

    // ASSERT
    Assert.That(result, Is.Not.Null);
}
```

Please notice how the second test:

* Returns a `Task` instead of `void`
* Is declared as an asynchronous method by the keyword `async`
* Awaits the execution of the `LoadAsync` method
* Is extremely easy to read while taking full advantage of the `async/await` functionality

