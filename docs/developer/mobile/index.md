
# NextNonce Mobile Documentation

Welcome to the official developer documentation for the NextNonce Mobile App. This application is the cross-platform client for the NextNonce Crypto Portfolio Tracker, running on both Android and iOS from a single, shared codebase.

This documentation provides a comprehensive guide for developers, covering the system's architecture, setup procedures, and core components.

### Documentation Sections

To get started, explore the following sections:

  - [**Installation Guide**](./installation.md): A step-by-step guide to setting up the mobile project on a local machine.
  - **Architecture Deep Dive**:
      - [**Overview**](./architecture/overview.md): A high-level look at the system's design and the principles of its Clean Architecture.
      - [**UI**](./architecture/ui.md): How the 100% shared UI is built with Compose Multiplatform.
      - [**Navigation**](./architecture/navigation.md): The type-safe navigation graph that powers screen transitions.
      - [**Result and Error Handling**](./architecture/result-error.md): The robust, exception-free error handling pattern.
      - [**Dependency Injection**](./architecture/di.md): The use of Koin for managing dependencies across the app.
      - [**Network**](./architecture/network.md): The Ktor-based networking layer and its authentication features.
      - [**Authentication**](./architecture/auth.md): The Supabase integration for user authentication and backend synchronization.
      - [**Reactive Data with Kotlin Flows**](./architecture/flow.md): The use of Kotlin Flow for creating a reactive, real-time UI.
      - [**Caching Strategy**](./architecture/caching.md): The multi-layered caching system designed for an optimal user experience.
      - [**Numerical Precision & Formatting**](./architecture/bignum.md): The approach to handling financial data with `BigDecimal` and platform-idiomatic formatting.

-----

## About the NextNonce Mobile App

The NextNonce mobile app is designed to be a simple, intuitive, and secure portfolio tracker perfect for both crypto beginners and regular users. It leverages the power of Kotlin Multiplatform to deliver a true native experience on Android and iOS without compromising on performance or code quality.

### Key Features

  - **Cross-Platform Excellence**: Built with **Kotlin Multiplatform** and a **100% shared UI** powered by **Compose Multiplatform**, ensuring feature parity and a consistent feel across devices.
  - **Completely Safe & Non-Custodial**: Security is paramount. The app only requires public wallet addresses, never private keys or seed phrases.
  - **Unified Asset View**: A standout feature that simplifies cross-chain tracking by aggregating popular tokens (like ETH, USDT) into a single, unified balance view.
  - **Broad Blockchain & Wallet Support**: Tracks assets across 15+ popular EVM blockchains and supports both standard **Simple** (EOA) and **Smart** wallets.
  - **Reactive & Real-Time**: Built on **Kotlin Flow**, the UI automatically updates to reflect the latest on-chain data, providing a dynamic and live experience.
  - **Robust Caching**: Implements a multi-layered caching strategy using both network and on-device database caches to provide an instant, responsive feel, even on slow networks.
  - **Secure Authentication**: Powered by **Supabase** for secure email/password and native Google Sign-In (Android).

### Technology Stack

  - **Core**: [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform-mobile-getting-started.html), [Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
  - **UI**: [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/), [Compose Navigation](https://developer.android.com/jetpack/compose/navigation)
  - **Architecture**: Clean Architecture, MVVM, [Koin](https://insert-koin.io/) for Dependency Injection
  - **Networking**: [Ktor Client](https://www.google.com/search?q=https://ktor.io/docs/client-overview.html)
  - **Database (Caching)**: [Room](https://developer.android.com/training/data-storage/room)
  - **Authentication**: [Supabase](https://supabase.com/)
  - **Image Loading**: [Kamel](https://github.com/Kamel-Media/Kamel)
  - **Logging**: [Kermit](https://kermit.touchlab.co/)
  - **Numerical Precision**: [BigNum](https://www.google.com/search?q=https://github.com/ionspin/kotlin-bignum)

### High-Level Architecture

The application follows the principles of **Clean Architecture**, strictly separating the code into three distinct layers:

1.  **Presentation Layer**: The UI, built entirely with Compose Multiplatform. It consists of `Screens` (Composables), `ViewModels`, and `State` objects to create a unidirectional data flow.
2.  **Domain Layer**: The core of the application. It contains all business logic, defined in `UseCases`, and the contracts for data handling, defined in `Repository` interfaces. This layer is pure Kotlin, independent of any platform or framework.
3.  **Data Layer**: Responsible for providing data to the domain layer. It contains the `Repository` implementations that fetch data from either the network (`RemoteDataSource`) or the local Room database (`LocalDataSource`).

This layered approach ensures the application is decoupled, highly testable, and easy to maintain and scale over time.