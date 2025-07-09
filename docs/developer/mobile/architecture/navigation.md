
Navigation in the NextNonce app is managed entirely within the shared `commonMain` source set using **Compose Navigation for Multiplatform** (`org.jetbrains.androidx.navigation:navigation-compose`). This modern library allows for a fully shared, type-safe navigation graph that is both robust and easy to maintain.

### Type-Safe Destinations

The core of the navigation architecture is its type-safety, which is achieved by defining every screen destination as a **serializable object or data class**. This eliminates error-prone string-based routes and ensures that all navigation actions are validated at compile time.

All possible navigation destinations are defined in a single file, `NextNonceScreens.kt`:

```kotlin
import kotlinx.serialization.Serializable

@Serializable
object Start

@Serializable
object Auth

@Serializable
object Home

@Serializable
data class AddWallet(val portfolioId: String)

@Serializable
data class Portfolio(val portfolioId: String)

@Serializable
data class Wallet(val walletId: String, val walletName: String? = null)
```

  - **`object`**: Used for screens that do not require any parameters, like `Home` or `Auth`.
  - **`data class`**: Used for screens that need arguments, such as `Portfolio`, which requires a `portfolioId`.

The `@Serializable` annotation from the `kotlinx.serialization` library is key to this approach, allowing the navigation library to handle the serialization and deserialization of these objects automatically.

### The Navigation Graph

The entire navigation flow of the application is defined within a single `NavHost` composable located in `App.kt`. This `NavHost` acts as the central hub, mapping each type-safe destination to its corresponding screen composable.

```kotlin
@Composable
fun App() {
    val navController: NavHostController = rememberNavController()
    NextNonceTheme {
        NavHost(
            navController = navController,
            startDestination = Start, // The journey starts here
            // ...
        ) {
            // Defines the screen for the 'Start' destination
            composable<Start> {
                StartScreen(
                    onUserSignedIn = {
                        // Type-safe navigation to the Home screen
                        navController.navigate(Home) {
                            popUpTo(Start) { inclusive = true } // Clears the back stack
                            launchSingleTop = true
                        }
                    },
                    // ...
                )
            }

            // Defines the screen for the 'Wallet' destination
            composable<Wallet> { navBackStackEntry ->
                // Safely extracts the Wallet object and its arguments
                val walletRoute = navBackStackEntry.toRoute<Wallet>()
                WalletScreen(
                    id = walletRoute.walletId,
                    name = walletRoute.walletName,
                    onBackClicked = {
                        navController.navigateUp()
                    }
                )
            }
            
            // ... other destinations
        }
    }
}
```

Key points of this implementation include:

  - **`NavHost`**: The container for the entire navigation graph. It's configured with a `navController` and a `startDestination`.
  - **`composable<T>()`**: This builder function links a serializable destination type (e.g., `composable<Start>`) to the UI content for that screen.
  - **Argument Passing**: For destinations with parameters (like `Wallet`), arguments are extracted in a type-safe manner using `navBackStackEntry.toRoute<Wallet>()`. This prevents runtime errors by ensuring the correct data type is always received.
  - **Back Stack Management**: Navigation actions often include logic to manage the back stack. For example, `popUpTo(Start) { inclusive = true }` is used after the initial authentication flow to remove the `Start` and `Auth` screens, preventing the user from navigating back to them after logging in.

### Platform-Conscious UI Patterns

To ensure the application feels intuitive and "native" on both Android and iOS without writing platform-specific code, we've adopted a consistent UI pattern for all secondary screens (i.e., any screen navigated to from a primary tab like `Home`).

This pattern is built around the `Scaffold` composable, which provides a standard layout structure. Every secondary screen implements a `topBar` to offer a clear and consistent navigation anchor for the user.

**Example `Scaffold` Implementation:**

```kotlin
Scaffold(
    topBar = {
        // A custom TopAppBar is used for all secondary screens
        AddWalletTopBar(
            onBackClicked = onBackClicked,
        )
    }
) { 
    // Screen content goes here
    // ...
}
```

#### The Top App Bar

The key to this pattern is a custom `CenterAlignedTopAppBar`. It is intentionally minimal to align with modern design trends and feel familiar to iOS users, while still adhering to Android's Material Design principles.

**Shared `TopAppBar` Code:**

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
private fun AddWalletTopBar(
    onBackClicked: () -> Unit
) {
    CenterAlignedTopAppBar(
        title = { /* Intentionally left empty for a minimal look */ },
        navigationIcon = {
            IconButton(onClick = onBackClicked) {
                Icon(
                    imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                    contentDescription = stringResource(Res.string.back),
                )
            }
        },
        colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
            containerColor = Color.Transparent
        )
    )
}
```

This implementation has two main goals:

1.  **Provide Clear Navigation**: The `navigationIcon` ensures a prominent back button is always present on the top-left of secondary screens. This is a universally understood pattern that allows users to easily navigate up the back stack.
2.  **Feel Natural on iOS**: The use of a top bar with a distinct back button, but without a title, mirrors common navigation patterns in many iOS applications. This helps iPhone users feel immediately at home in the app, without creating a layout that looks out of place on Android.