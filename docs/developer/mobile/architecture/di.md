
Dependency Injection in the NextNonce application is managed by **Koin**, a pragmatic and lightweight DI framework for Kotlin. Koin is leveraged across all layers of the architecture—`data`, `domain`, and `presentation`—to provide dependencies, manage object lifecycles, and decouple components. The entire DI setup is configured in the shared `commonMain` source set, making it consistent across both Android and iOS.

### Modular Configuration

The dependency graph is organized into several logical, feature-based modules. This modular approach keeps the DI setup clean, scalable, and easy to navigate. Each feature, such as `auth`, `portfolio`, or `wallet`, has its own Koin module file that declares its specific dependencies.

All these modules are aggregated and initialized in a central `initKoin` function, which is called once when the application starts.

**`Module.kt`**

```kotlin
// This function is the main entry point for starting Koin.
fun initKoin(config: KoinAppDeclaration? = null) =
    startKoin {
        config?.invoke(this)
        modules(
            platformModule,
            authModule,
            startModule,
            coreModule,
            userModule,
            walletModule,
            portfolioModule,
            homeModule,
        )
    }

// The common module expects a platform-specific module.
expect val platformModule: Module
```

A key pattern here is the `expect val platformModule: Module`. This allows the shared `commonMain` code to depend on a module that will be provided by each specific platform (`androidMain` and `iosMain`). This is how platform-specific dependencies, like database builders or HTTP client engines, are injected into the shared graph.

### Declaring Dependencies

Dependencies are declared using Koin's simple DSL. The project primarily uses:

  - **`singleOf(::ClassName)`**: The most common declaration, used for creating singleton instances. It's a concise way to provide a class and have Koin resolve its dependencies automatically. This is used for `UseCase` and `Repository` implementations.
  - **`.bind<Interface>()`**: Used in conjunction with `singleOf` to bind a concrete implementation to its interface (e.g., `singleOf(::AuthRepositoryImpl).bind<AuthRepository>()`). This allows other classes to depend on the abstract interface, promoting better decoupling.
  - **`viewModel { ... }`**: A special builder for declaring `ViewModel` instances. Koin provides special handling for ViewModels to align with their lifecycle.

**Example: `portfolioModule`**

```kotlin
val portfolioModule = module {
    // Repositories and DataSources are singletons
    singleOf(::PortfolioRepositoryImpl).bind<PortfolioRepository>()
    singleOf(::RemotePortfolioDataSourceImpl).bind<RemotePortfolioDataSource>()

    // UseCases are also singletons
    singleOf(::CreatePortfolioWalletUseCase)
    singleOf(::GetDefaultPortfolioUseCase)

    // ViewModel declaration
    viewModel { (portfolioId: String) ->
        PortfolioViewModel(
            id = portfolioId,
            get(), // Koin injects the GetPortfolioWalletsUseCase
            get(), // Koin injects the GetPortfolioTotalBalanceUseCase
            get(), // Koin injects the GetPortfolioAssetBalancesUseCase
        )
    }
}
```

### ViewModels with Parameters

A powerful feature used in the app is the ability to inject ViewModels that require runtime parameters. For screens that need an ID from the navigation graph (like the `PortfolioScreen` or `WalletScreen`), the `viewModel` is declared with parameters.

`viewModel { (portfolioId: String) -> PortfolioViewModel(...) }`

This syntax allows Koin to create a `ViewModel` factory. When the `ViewModel` is requested in the UI, the runtime parameter (`portfolioId`) can be passed in, while Koin automatically resolves and injects the rest of the dependencies (`UseCases`, etc.). This provides a clean solution for creating stateful ViewModels that depend on both runtime data and injected services.