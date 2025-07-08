# Welcome to NextNonce Docs

Welcome to the central documentation hub for NextNonce, your on-chain crypto portfolio tracker. This site provides comprehensive information for users, developers, and anyone interested in the NextNonce ecosystem.

## What is NextNonce?

NextNonce is a crypto portfolio tracker designed for simplicity and power. It allows you to monitor your assets across a wide range of blockchains without ever needing to expose private keys or seed phrases. Provide your public wallet address, and let NextNonce do the rest.

The ecosystem consists of two main components:

* **NextNonce Mobile App**: A cross-platform Android and iOS application built with Kotlin Multiplatform, providing a clean and intuitive interface to track your portfolio on the go.
* **NextNonce Backend**: A robust and scalable backend service built with NestJS that powers the mobile application by fetching, aggregating, and serving blockchain data efficiently.

## ✨ Core Features

The entire NextNonce platform is built around a set of powerful, user-centric features:

* **🌐 Broad Blockchain Support:** Track your assets across **15+** of the most popular EVM blockchains, including Ethereum, BNB Chain, Polygon, Arbitrum, Optimism, and many more. Support for additional chains is continuously being added.
* **💎 Unified Asset View:** A standout feature that simplifies cross-chain tracking. NextNonce automatically groups popular tokens (like `ETH,` `USDC,` `USDT,` and `BNB`) from all supported networks into a unified view. See the total value of each asset, and then tap to see a detailed breakdown of your balance on each individual network.
* **👛 Universal Wallet Support:** Whether you have **Simple** or **Smart** wallets, NextNonce will fetch and display the balances accurately.
* **🚀 High Performance:** Leveraging a sophisticated caching layer with Redis on the backend and Room on mobile with a robust provider-based architecture, the app delivers data with low latency for a smooth and responsive user experience.
* **🔒 Completely Safe & Non-Custodial:** Your security is a top priority. NextNonce **never** asks for private keys or seed phrases. The app works by tracking public wallet addresses only, ensuring your assets are always under your control.
* **🔐 Secure Authentication:** User accounts and API access are secured using modern standards. The mobile app features easy sign-in with Email or Google (powered by Supabase), and all backend endpoints require a valid JWT, ensuring that data access is always protected.

## 🚀 Get Started

Whether you're a user looking to track a portfolio or a developer interested in the technology, here's how to dive in:

### 👤 For Users

Ready to start tracking your assets? The User Guide has everything you need to know.

* **[User Guide](https://docs.nextnonce.com/user/)**
    > Learn how to use the app, add wallets, and make the most of every feature.

* **[Download the App](https://github.com/NextNonce/Mobile/releases)**
    > Get the latest version for Android directly from the GitHub releases.

### 🧑‍💻 For Developers

Are you interested in architecture or want to contribute to it? The developer docs provide a deep dive into the tech stack and codebase for both the mobile and backend applications.

* **[Mobile Developer Docs](https://docs.nextnonce.com/developer/mobile/)**
    > Explore the Kotlin Multiplatform codebase, Compose Multiplatform UI, and the clean MVVM architecture of the iOS & Android app.

* **[Backend Developer Docs](https://docs.nextnonce.com/developer/backend/)**
    > Discover the NestJS framework, the provider-based architecture, API specifications, and how data fetching and caching are handled.