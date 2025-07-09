Of course. The reactive nature of the application, built on Kotlin's `Flow`, is a cornerstone of its architecture. Here is the documentation for this section.


The NextNonce application is built around a reactive architecture, using **Kotlin `Flow`** as a first-class citizen for managing asynchronous data streams. Unlike traditional one-shot data requests, `Flow` allows us to create streams of data that the UI can observe. When the underlying data changes, the UI automatically updates, providing a seamless and real-time user experience.

This approach is used for nearly all data that is displayed to the user, ensuring consistency across different screens and reducing redundant data fetching.

### Flows in the Repository Layer

The repository layer is the primary source of truth for these data streams. Functions that provide data for continuous observation return a `Flow`, while one-time actions (like creating an item) are simple `suspend` functions.

**`PortfolioRepository.kt` Interface Example:**

```kotlin
interface PortfolioRepository {
    // Returns a stream of portfolio lists that updates on changes.
    fun portfoliosFlow(): Flow<Result<List<PortfolioModel>, DataError>>

    // A one-time action that returns a single result.
    suspend fun createPortfolioWallet(
        id: String,
        command: CreatePortfolioWalletCommand
    ): EmptyResult<DataError>

    // Returns a stream of wallets for a specific portfolio.
    fun portfolioWalletsFlow(
        id: String
    ): Flow<Result<List<PortfolioWalletModel>, DataError>>
}
```

### The Trigger & Refresh Pattern

To ensure data is always fresh, we use a pattern where one-shot actions can "trigger" a refresh on an active `Flow`. This is achieved using a `MutableSharedFlow` that acts as a manual refresh signal.

When a one-shot action like `createPortfolioWallet` completes successfully, it emits a `Unit` value to a trigger `Flow`. Any collector observing a `Flow` that listens to this trigger will automatically refetch its data.

**Implementation Example:**

```kotlin
// In PortfolioRepositoryImpl

private val refreshWallets = mutableMapOf<String, MutableSharedFlow<Unit>>()

private fun triggerFor(id: String) =
    refreshWallets.getOrPut(id) { MutableSharedFlow(replay = 1) }

// The one-shot action emits a value to the trigger.
override suspend fun createPortfolioWallet(id: String, ...): EmptyResult<DataError> {
    return when (val res = remoteDataSource.createPortfolioWallet(id, dto)) {
        is Result.Success -> {
            triggerFor(id).emit(Unit) // This notifies observers to refresh.
            Result.Success(Unit)
        }
        is Result.Error -> res
    }
}

// The Flow is built to listen to the trigger.
@OptIn(ExperimentalCoroutinesApi::class)
override fun portfolioWalletsFlow(id: String): Flow<Result<List<PortfolioWalletModel>, DataError>> {
    return triggerFor(id)
        .onStart { emit(Unit) } // Fetch immediately on first collection.
        .flatMapLatest { // Re-fetch whenever the trigger emits.
            // ... logic to fetch wallets from remoteDataSource ...
        }
}
```

This pattern ensures that when a user adds a wallet on one screen and navigates back, the list on the previous screen is already up-to-date without any complex manual intervention.

### Advanced Shared Flows for Efficiency

For data that might be observed by multiple screens simultaneously (like a portfolio's total balance), we use a more advanced pattern to avoid redundant work. The `cachedBalancesFlow` is a prime example.

This flow is designed to be created **only once per portfolio ID** and shared among all collectors. It uses a `Mutex` to prevent race conditions during creation and employs `shareIn` to convert the "cold" flow into a "hot" `SharedFlow`.

  - **Smart Polling**: It implements a polling loop that repeatedly fetches data from the backend until the data is marked as "actual," with an increasing delay between attempts.
  - **Resource Efficiency**: The `shareIn` operator is configured with `SharingStarted.WhileSubscribed(5000L)`, which means the polling stops 5 seconds after the last screen stops observing it, saving battery and network resources. The `replay = 1` argument ensures new screens get the latest data immediately upon subscription.

This ensures that two different screens observing the same balance will always see the exact same, synchronized data while only maintaining a single upstream connection.

### Combining Flows in the ViewModel

The true power of this reactive architecture is realized in the `ViewModel`, where multiple streams can be combined into a single, cohesive state for the UI.

For example, to display a list of wallets, each with its own real-time balance, we can `combine` multiple flows:

```kotlin
// In HomeViewModel
val balanceFlows = walletModels.map { wallet ->
    getWalletTotalBalanceUseCase.execute(wallet.id)
}

// The 'combine' operator takes the latest value from each individual
// balance flow and merges them into a single list for the UI.
val combinedBalancesFlow = combine(balanceFlows) { latestBalances ->
    // ... logic to map wallets and their balances to UI items ...
}

// Emit a placeholder list first for a better UX.
combinedBalancesFlow.onStart { emit(initialPlaceholderList) }
```

This pattern is incredibly powerful, allowing for a complex UI that is always up-to-date, with minimal and highly efficient data fetching.