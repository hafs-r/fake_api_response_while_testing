
## How to implement a Fake offline API to make android testing easy?

Most of the times when you code an application which consumes REST API then writing tests, especially UI tests is not that easy. We will have to set up many mock test responses, and when we have many API calls it becomes really overwhelming. We were working on an android project last time and we implemented fake api response to make testing less complex.

![Banner](images/banner.png)

I will simply explain the solution we followed to improve the stability and clarity of the test by implementing offline API responses by intercepting OkHttpClient.

Hope you already understand what is OkHttpClient, Retrofit, and the use of Interceptor, etc.

I’ve implemented a FakeInterceptor from which I will redirect API request with a response file with JSON contents. To mock all scenarios some times I read the request parameter from the request object and change response accordingly.
Offline Fake API reduces a lot of complexity we face if we are testing with a test API environment or making a mock remote API or mocking with Mockito or any other tool.

**FakeInterceptor.kt** 
```
import android.content.Context
import com.apptualizer.fakeapi.helper.ResourcesHelper
import com.apptualizer.fakeapi.helper.ResponseHelper
import okhttp3.*
import java.io.IOException

class FakeInterceptor @JvmOverloads constructor(val context: Context) : Interceptor {

    @Throws(IOException::class)
    override fun intercept(chain: Interceptor.Chain): Response {
        val url = chain.request().url()
        val path = ResponseHelper.mapProperResponse(url.encodedPath(), chain.request())
        val outPutString = ResourcesHelper.loadFileAsString(path)

        return Response.Builder()
                .code(ResponseHelper.errorCode)
                .message(outPutString)
                .request(chain.request())
                .protocol(Protocol.HTTP_2)
                .body(ResponseBody.create(
                  MediaType.parse("application/json"),outPutString))
                .addHeader("content-type", "application/json").build()
    }

}
```

In the above code, there are two other classes ResourcesHelper and ResponseManager are added below.

ResourceHelper is a helper class that reads the JSON file placed in the asset folder from the given path and returns the JSON content in string format.

**ResourcesHelper.kt** 
```
import android.util.Log
import androidx.test.platform.app.InstrumentationRegistry
import java.io.BufferedReader
import java.io.IOException
import java.io.InputStreamReader
object ResourcesHelper {
  
  fun loadFileAsString(path: String): String? {
        var bufferedReader: BufferedReader? = null
        try {
            val stringBuilder = StringBuilder()
            bufferedReader = BufferedReader(
              InputStreamReader(InstrumentationRegistry.getInstrumentation()
                    .context.assets.open(path)))
            var isFirst = true
            var line: String?
      do {
              line = bufferedReader?.readLine()
                if (line == null)
                    break
                if (isFirst)
                    isFirst = false
                else
                    stringBuilder.append('\n')
                stringBuilder.append(line)
            } while (true)
      return stringBuilder.toString()
        } catch (e: Exception) {
            Log.e("FakeInterceptor", e.message.plus("> Error opening asset $path."))
        } finally {
            if (bufferedReader != null) {
                try {
                    bufferedReader?.close()
                } catch (e: IOException) {
                    Log.e("FakeInterceptor", e.message.plus("> Error closing  $path."))
                }
            }
        }
        return null
    } 
    
}
```


ResponseManager is a class that manages the response as we need mapping the URL to a proper response file is the main duty of the Response manager.

**ResponseManager.kt**
```
import okhttp3.Request
import okio.Buffer

object ResponseManager {

    var next_match = true
    var errorCode = 200

    private fun bodyToString(request: Request): String {

        try {
            val copy = request.newBuilder().build()
            val buffer = Buffer()
            copy.body()!!.writeTo(buffer)
            return buffer.readUtf8()
        } catch (e: Exception) {
            return "did not work".plus(e.message)
        }

    }

    fun mapProperResponse(input: String, request: Request): String {

        var output = input
        if (input.endsWith("xxxxxx.json")) {
            output = "api/xxxxx.json"
        } else if (input.endsWith("xxxxx")) {
            output = "api/xxxxx.json"
        } else if (input.endsWith("login")) {

            var bodyAsString = bodyToString(request)
            bodyAsString = bodyAsString.substring(bodyAsString.indexOf("xxxxxx\":") + xx)
            val xxxxx = bodyAsString.substring(0, bodyAsString.indexOf("\""))
            bodyAsString = bodyAsString.substring(bodyAsString.indexOf("xxxxx\":") + xx)
            val xxxxx = bodyAsString.substring(0, bodyAsString.indexOf("\""))

            if (xxxxx.equals("xxx@xxxx.xxx")) {
                errorCode = xxx
                output = "api/xxxx/xxxxx_xxx.json"
            } else {
                errorCode = 200
                output = "api/accounts/login.json"
            }
        }

        return output
    }
}
```

I’m using Dagger to inject the dependencies but you can follow any dependency injection methods that you follow.

**dagger_module.kt**
```
@Provides
@Singleton
internal fun provideOkhttpClient(
                 cache: Cache,
                 networkMonitor:NetworkMonitor,
                 @ApplicationContext context: Context,
                 eventBus: RxEventBus, 
                 deviceType: CurrentDeviceType): OkHttpClient {
      val client = OkHttpClient.Builder()
      .cache(cache)
      .connectTimeout(CONNECTION_TIME_OUT.toLong(),TimeUnit.SECONDS)
      .readTimeout(READ_TIME_OUT.toLong(), TimeUnit.SECONDS)
      .addNetworkInterceptor(TimberLoggingInterceptor())
      .addInterceptor(FakeInterceptor(context))
     
      return client.build()
}
@Provides
@Singleton
internal fun provideRetrofit(
                 gson: Gson, 
                 okHttpClient: OkHttpClient): Retrofit {
      val restAdapterBuilder = Retrofit.Builder()
      .client(okHttpClient)
      .baseUrl(Constants.API_BASE_URL)
      .addConverterFactory(GsonConverterFactory.create(gson))
      .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
      
      return restAdapterBuilder.build()
}
```

Hope it will help to make your life easy, please comment incase anything to be updated. Thank you.

