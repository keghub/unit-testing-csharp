# When-to-test

Whilst there is an intrinsic value in having a test suite, especially when using typed languages like C\#, unit tests express their true value when executed and given the possibility of actually failing.

Since unit tests are disconnected from real data and real dependencies, their output should not vary after the application has been packaged and, eventually, deployed.

This means that there are two times in the lifecycle of the application where executing tests brings the most value:

* When the library/application is being written
* When the library/application is being packaged

Continuously running the tests while the application or library is being written helps the developers keeping aligned the expected behavior of the application and the actual one.

Ideally, Developers should manually start the execution of the test suite after any change. Modern IDEs like the Visual Studio 2019 or JetBrains ReSharper support background execution of unit tests immediately after changes are saved.

In any case, developers should always run the test suite before pushing their changes to the code repository.

Once the changes have been pushed to the code repository, it is a good practice to have the component being automatically built and put through the set of unit tests.

In organizations using repositories based on multiple branches, unit tests should be executed both before and after the changes being merged in the main branch.

