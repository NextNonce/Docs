

A core principle of the **NextNonce** architecture is robust and predictable error handling. Instead of relying on traditional exceptions for control flow, the application employs a type-safe, explicit error propagation system inspired by Rust. This ensures that any operation that can fail must be handled explicitly, preventing unexpected runtime crashes and making the data flow easier to reason about.

### The `DataError` Type

At the heart of this system is the `DataError` sealed interface. This type represents all possible, known errors that can occur within the data layer of the application. By defining errors in a structured hierarchy, we can handle them with precision.

`DataError` is split into two categories: `Remote` for network-related issues and `Local` for on-device problems.

**`DataError.kt`**

```kotlin
package com.nextnonce.app.core.domain

sealed interface DataError : Error {

    /**
     * Represents remote data errors (network, server, etc.).
     */
    enum class Remote : DataError {
        NO_INTERNET,
        SERIALIZATION,
        UNAUTHORIZED,
        FORBIDDEN,
        NOT_FOUND,
        // ... and other remote errors
    }

    /**
     * Represents local data errors (disk, session, etc.).
     */
    enum class Local : DataError {
        DISK_FULL,
        NO_SESSION,
        UNKNOWN
    }
}
```

### The Custom `Result` Wrapper

All operations that can fail (e.g., network requests, database queries) do not throw exceptions. Instead, they return a custom `Result` object. This is a generic sealed interface that encapsulates either a successful outcome or a failure.

This custom implementation is preferred over the standard Kotlin `Result` because it allows for a custom `Error` type, enabling the use of our structured `DataError` system.

**`Result.kt`**

```kotlin
package com.nextnonce.app.core.domain

sealed interface Result<out D, out E : Error> {
    data class Success<D>(val data: D) : Result<D, Nothing>
    data class Error<E : com.nextnonce.app.core.domain.Error>(val error: E) : Result<Nothing, E>
}

// Helper extension functions like .map, .onSuccess, .onError are also provided
```

### Consistent Propagation Through Layers

The `Result` type is used as the return type for failable functions throughout all layers of the architecture, creating a clear and predictable data flow.

1.  **Data Source Layer**: The lowest layer initiates the pattern.

```kotlin
// RemotePortfolioDataSource.kt
interface RemotePortfolioDataSource  {
    suspend fun getPortfolios(): Result<List<PortfolioDto>, DataError.Remote>
    // ...
}
```

2.  **Repository Layer**: The repository consumes the `Result` from the data source. It uses a `when` expression, which is exhaustive, forcing the developer to handle both the `Success` and `Error` cases.

    ```kotlin
    // PortfolioRepositoryImpl.kt
    override suspend fun getWalletById(walletId: String): Result<WalletModel, DataError> {
        return when (val result = remoteDataSource.getWallet(walletId)) {
            is Result.Success -> Result.Success(result.data.toWalletModel()) // Map DTO to Domain Model
            is Result.Error -> result // Pass the error up the chain
        }
    }
    ```

This pattern continues up through the `UseCase` layer to the `ViewModel`, ensuring that any error is explicitly passed and handled at each step.

### Displaying Errors in the UI

The final step is to translate these structured `DataError` types into user-friendly messages. This is accomplished with a simple extension function, `toUIText()`, which maps each specific error case to a `StringResource`.

This decouples the presentation layer from the underlying error logic. The UI doesn't need to know what `DataError.Remote.FORBIDDEN` means; it only needs to display the corresponding text.

```kotlin
// DataErrorToString.kt
fun DataError.toUIText(): StringResource {
    val stringRes = when (this) {
        is DataError.Local -> when (this) {
            DataError.Local.NO_SESSION -> Res.string.error_no_session
            // ...
        }
        is DataError.Remote -> when (this) {
            DataError.Remote.NO_INTERNET -> Res.string.error_no_internet
            DataError.Remote.EMAIL_ALREADY_REGISTERED -> Res.string.error_account_exists
            // ...
        }
    }
    return stringRes
}
```

In the `ViewModel`, this function is called within the error handling block to set a user-readable error message in the UI state, completing the full, type-safe error handling lifecycle.