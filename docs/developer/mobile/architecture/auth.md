

Authentication in the NextNonce app is handled by **Supabase**, leveraging the powerful community-driven `supabase-auth-kt` library. This provides a secure and reliable way to manage user identity. The implementation supports both traditional email/password sign-up and sign-in, as well as a seamless native Google Sign-In experience on Android.

### Secure Configuration

All sensitive credentials required for authentication are managed securely and kept out of version control using the **`buildkonfig`** Gradle plugin.

The **`AuthProvider`** singleton object is the central point for initializing the Supabase client. It reads the `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `GOOGLE_WEB_CLIENT_ID` from the generated `BuildKonfig` file.

**`AuthProvider.kt`**

```kotlin
object AuthProvider {
    lateinit var client: SupabaseClient
        private set

    fun init() {
        client = createSupabaseClient(
            BuildKonfig.SUPABASE_URL,
            BuildKonfig.SUPABASE_ANON_KEY
        ) {
            install(Auth)
            install(ComposeAuth) {
                // Ensure Google Native Login is only configured on Android
                if (getPlatform().name == "Android") {
                    googleNativeLogin(BuildKonfig.GOOGLE_WEB_CLIENT_ID)
                }
            }
        }
    }
}
```

A key detail is the platform check within the `ComposeAuth` installer. This ensures that the native Google Sign-In flow is only initialized and available on the Android platform, preventing runtime errors on iOS.

### Architectural Flow & Backend Sync

The authentication logic follows the standard `ViewModel` -\> `UseCase` -\> `Repository` pattern. However, a crucial architectural decision was made to separate the authentication provider (Supabase) from the application's own backend user database.

When a new user registers, a two-step process occurs:

1.  **Supabase Authentication**: The user is first created in the Supabase authentication system.
2.  **Backend Registration**: Upon successful authentication, a call is made to the NextNonce backend via the `CreateUserUseCase` to create a corresponding user record in our own database.

This orchestration is handled within the `SignUpWithEmailUseCase` and `SignInWithGoogleUseCase`.

**`SignUpWithEmailUseCase.kt` Example**

```kotlin
class SignUpWithEmailUseCase(
    private val repo: AuthRepository,
    private val createUserUseCase: CreateUserUseCase
) {

    suspend fun execute(email: String, password: String): Result<AuthUserModel, DataError> {
        return when (val authUser = repo.signUpWithEmail(email, password)) {
            is Result.Success -> {
                when (val user = createUserUseCase.execute(authUser.data.toUserModel())) {
                    is Result.Success -> authUser // User creation was successful, return authUser
                    is Result.Error -> {
                        // If user creation failed, we can log the error and return it
                        AppLogger.e { "Failed to create user after sign up: ${user.error}" }
                        Result.Error(user.error)
                    }
                }
            }
            is Result.Error -> Result.Error(authUser.error)
        }
    }
}
```

This ensures that our application's user data stays synchronized with the third-party authentication provider.

### UI Implementation (`AuthScreen`)

The `AuthScreen` is the user-facing component for all authentication flows. It is only shown when the app cannot find an active Supabase session.

  - It uses the `rememberSignInWithGoogle` composable function from the Supabase library to provide a seamless and native sign-in experience on Android.
  - It uses a `LaunchedEffect` keyed to the sign-in/sign-up state. This ensures that the navigation (`onDone()`) away from the `AuthScreen` happens reliably and exactly once upon successful authentication.

<!-- end list -->

```kotlin
// AuthScreen.kt

// ...
val viewModel = koinViewModel<AuthViewModel>()
val state by viewModel.state.collectAsStateWithLifecycle()

// Navigate exactly once when sign‑in/up completes
LaunchedEffect(state.isSignIn || state.isSignUp) {
    if (state.isSignIn || state.isSignUp) onDone()
}
// ...
```