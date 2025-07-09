
In a financial application like NextNonce, user experience during data loading is critical. While our backend is optimized, it has limitations due to the limitations of different providers. Network latency also makes the situation worse. Showing a user a zero balance or a loading spinner while their data is being fetched can be alarming, creating the false impression that their funds are gone.

To prevent this and provide a stable, responsive experience, the application employs a multi-layered caching strategy. The core philosophy is: **it is always better to show the user their own slightly outdated data instantly, rather than showing them nothing.** This approach ensures the app feels fast and familiar, even before the latest real-time data has arrived.

### Network & Image Caching

The first layer of caching happens at the network level for non-critical, repeatable data.

  - **HTTP Caching**: The Ktor `HttpClient` is configured with the `HttpCache` plugin. This provides a standard, out-of-the-box cache for general API responses, reducing redundant network calls for data that doesn't change frequently.
  - **Image Caching**: `Kamel`, our image loading library, is configured with a robust two-tiered cache.
    ```kotlin
    val nnKamelConfig = KamelConfig {
        takeFrom(KamelConfig.Default)
        // In-memory cache for instant access
        imageBitmapCacheSize = KAMEL_CACHE_SIZE_MULTIPLIER * DefaultCacheSize

        // On-disk cache for persistence
        httpUrlFetcher {
            httpCache(size = KAMEL_CACHE_SIZE_MULTIPLIER * DefaultHttpCacheSize)
        }
    }
    ```
    This ensures that token and chain logos are loaded from the network only once and are then served instantly from memory or disk on subsequent views.

### Persistent On-Device Caching with Room

The most critical caching layer is the persistent on-device cache, powered by **Room**. This layer is responsible for storing user balance data directly on their device, enabling the "cache-then-network" strategy.

The setup is straightforward, using a `DAO` (Data Access Object) and an `Entity`. The `WalletBalancesDto` from the network is serialized into a JSON string and stored in the database.

**`WalletBalancesCacheDao.kt`**

```kotlin
@Dao
interface WalletBalancesCacheDao {
    @Query("SELECT * FROM WalletBalancesCacheEntity WHERE walletId = :walletId")
    suspend fun getBalancesCache(walletId: String): WalletBalancesCacheEntity?

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun upsertBalancesCache(entity: WalletBalancesCacheEntity)
}
```

**`WalletBalancesCacheEntity.kt`**

```kotlin
@Entity
data class WalletBalancesCacheEntity(
    @PrimaryKey
    val walletId: String,
    val timestamp: Long,
    val balancesJson: String // The DTO is stored as a JSON string
)
```

### The "Cache-Then-Network" Flow

This strategy is implemented within the `WalletRepositoryImpl`. When a `Flow` for wallet balances is requested, it follows a clear, user-centric process:

1.  **Emit Cached Data First**: The `flow` builder *immediately* queries the Room database for a cached `WalletBalancesCacheEntity`. If one is found, its data is instantly converted and emitted to the UI. This is the key step that prevents the user from seeing an empty or zero-balance screen.

2.  **Fetch Fresh Data in the Background**: After emitting the cached data, the flow proceeds to fetch fresh data from the `remoteDataSource` inside a loop. This ensures the UI is always subscribed to the latest updates.

3.  **Update UI and Cache**: When fresh data is successfully fetched, it is:  
    a. Emitted to the `Flow`, causing the UI to update with the latest information.  
    b. Saved back into the Room database using `upsertBalancesCache`, ensuring the cache is fresh for the next time the app is launched.

**`WalletRepositoryImpl.kt` Implementation**

```kotlin
private fun createAndShareBalancesFlow(
    walletId: String
): SharedFlow<Result<WalletBalancesModel, DataError>> = flow {
    // 1. Emit cached data immediately if it exists.
    val cached = walletBalancesCacheDao.getBalancesCache(walletId)
    if (cached != null) {
        AppLogger.d { "Using cached balances for wallet $walletId" }
        emit(Result.Success(cached.toDto().toWalletBalancesModel()))
    }

    // ...

    // 2. Start fetching fresh data in a loop.
    while (true) {
        when (val dtoResult = remoteDataSource.getBalances(wallet.address)) {
            is Result.Success -> {
                val dto = dtoResult.data
                // 3a. Emit fresh data to the UI.
                emit(Result.Success(dto.toWalletBalancesModel()))

                // 3b. Update the on-device cache.
                walletBalancesCacheDao.upsertBalancesCache(
                    WalletBalancesCacheEntity(walletId, now(), dto)
                )
            }
            is Result.Error -> emit(Result.Error(dtoResult.error))
        }
        randomDelay() // Avoid hammering the server.
    }
}.shareIn(...)
```

While this means the user might see slightly outdated data for a brief moment, this trade-off is explicitly chosen to provide a more stable and reassuring user experience.