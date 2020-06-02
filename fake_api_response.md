
##How to implement a Fake offline API to make android testing easy?

Most of the times when you code an application which consumes REST API then writing tests, especially UI tests is not that easy. We will have to set up many mock test responses, and when we have many API calls it becomes really overwhelming. We were working on an android project last time and we implemented fake api response to make testing less complex.

![Banner](images/banner.png)

I will simply explain the solution we followed to improve the stability and clarity of the test by implementing offline API responses by intercepting OkHttpClient.

Hope you already understand what is OkHttpClient, Retrofit, and the use of Interceptor, etc.

I’ve implemented a FakeInterceptor from which I will redirect API request with a response file with JSON contents. To mock all scenarios some times I read the request parameter from the request object and change response accordingly.
Offline Fake API reduces a lot of complexity we face if we are testing with a test API environment or making a mock remote API or mocking with Mockito or any other tool.

<script src="https://gist.github.com/hafsalrahman/53037e40d86fafd5ea560994033bb388.js"> </script>

In the above code, there are two other classes ResourcesHelper and ResponseManager are added below.

ResourceHelper is a helper class that reads the JSON file placed in the asset folder from the given path and returns the JSON content in string format.

<script src="https://gist.github.com/hafsalrahman/de0992b0d3033874cf66fd30d23184be.js"> </script>

ResponseManager is a class that manages the response as we need mapping the URL to a proper response file is the main duty of the Response manager.

<script src="https://gist.github.com/hafsalrahman/a868c3bc12a8be4118d51042aa814379.js"> </script>

I’m using Dagger to inject the dependencies but you can follow any dependency injection methods that you follow.

<script src="https://gist.github.com/hafsalrahman/18bd6b7a222cfcedd1ea082c5ae2e26e.js"></script>

Hope it will help to make your life easy, please comment incase anything to be updated. Thank you.

