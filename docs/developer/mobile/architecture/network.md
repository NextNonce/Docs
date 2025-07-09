

The networking layer in NextNonce is built entirely on **Ktor**, a modern, asynchronous, and multiplatform networking framework from JetBrains. This choice ensures that all network-related logic is written once in the `commonMain` source set and works seamlessly across both Android and iOS.

### HttpClient Configuration

The central piece of the networking setup is the `HttpClientFactory`. This object is responsible for creating and configuring the `HttpClient` instance used for all backend API communications. The client is configured with a suite of powerful Ktor plugins to handle common networking tasks automatically.

**`HttpClientFactory.kt`**

```kotlin
object HttpClientFactory {
    fun createBackend(
        engine: HttpClientEngine,
        getBearerTokensUseCase: GetBearerTokensUseCase,
        refreshBearerTokensUseCase: RefreshBearerTokensUseCase
    ): HttpClient {
        return HttpClient(engine) {
            // 1. JSON Serialization
            install(ContentNegotiation) {
                json(JsonHumanReadable)
            }
            // 2. Automatic Authentication
            install(Auth) {
                bearer {
                    loadTokens { getBearerTokensUseCase.execute() }
                    refreshTokens { refreshBearerTokensUseCase.execute() }
                }
            }
            // 3. Timeouts
            install(HttpTimeout) {
                socketTimeoutMillis = SOCKET_TIMEOUT_MILLIS
                requestTimeoutMillis = REQUEST_TIMEOUT_MILLIS
            }
            // 4. Caching and Logging
            install(HttpCache)
            install(Logging) {
                logger = KermitLoggerAdapter
                level = LogLevel.ALL
            }
            // 5. Default Request Configuration
            defaultRequest {
                url(API_BASE_URL)
                contentType(ContentType.Application.Json)
            }
        }
    }
}
```

Key plugins and configurations include:

1.  **`ContentNegotiation`**: Automatically serializes outgoing data to JSON and deserializes incoming JSON responses using a custom `JsonHumanReadable` configuration. This configuration is set up to handle complex types like `BigDecimal` and polymorphic classes.
2.  **`Auth`**: Manages authentication transparently. It automatically attaches bearer tokens to outgoing requests using the `GetBearerTokensUseCase`. Crucially, it's also configured to automatically refresh expired tokens using the `RefreshBearerTokensUseCase` and retry the original request, simplifying token management throughout the app.
3.  **`HttpTimeout`**: Enforces strict request and socket timeouts to prevent the app from hanging on slow network connections.
4.  **`HttpCache` & `Logging`**: Provides out-of-the-box response caching and detailed request/response logging (using Kermit) for easier debugging.
5.  **`defaultRequest`**: Sets the base URL and `Content-Type` header for all requests, reducing boilerplate code in the data sources.

### Robust Error Handling: `safeCall`

To ensure robust and predictable error handling, all network requests made by the data sources are wrapped in a `safeCall` function. This function is the cornerstone of the networking layer's resilience.

**`HttpClientExt.kt`**

```kotlin
suspend inline fun <reified T> safeCall (
    execute: () -> HttpResponse
): Result<T, DataError.Remote> {
    val response = try {
        execute()
    } catch (e: SocketTimeoutException) {
        return Result.Error(DataError.Remote.REQUEST_TIMEOUT)
    } catch (e: UnresolvedAddressException) {
        return Result.Error(DataError.Remote.NO_INTERNET)
    } // ... other exceptions

    return responseToResult(response)
}
```

The `safeCall` function's responsibilities are:

1.  **Catch Low-Level Exceptions**: It catches common network exceptions like timeouts or connectivity issues and maps them directly to the appropriate `DataError.Remote` type.
2.  **Delegate to `responseToResult`**: If the request succeeds without an exception, it passes the `HttpResponse` to the `responseToResult` helper function.

This second function inspects the HTTP status code and converts the response into the application's standard `Result` type.

```kotlin
suspend inline fun <reified T> responseToResult(
    response: HttpResponse
): Result<T, DataError.Remote> {
    return when (response.status.value) {
        in 200..299 -> {
            try {
                Result.Success(response.body<T>())
            } catch (e: Exception) {
                Result.Error(DataError.Remote.SERIALIZATION)
            }
        }
        401 -> Result.Error(DataError.Remote.UNAUTHORIZED)
        404 -> Result.Error(DataError.Remote.NOT_FOUND)
        // ... other status codes
        else -> Result.Error(DataError.Remote.UNKNOWN)
    }
}
```

This pattern ensures that every network call returns a `Result<T, DataError.Remote>`, forcing the calling code (in repositories and use cases) to explicitly handle both success and failure scenarios, leading to a more stable and predictable application.