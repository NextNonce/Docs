# NextNonce Backend Documentation

Welcome to the official developer documentation for the NextNonce backend. This backend is the server-side engine that powers the NextNonce Crypto Portfolio Tracker application, handling everything from data aggregation and user authentication to balance calculations and portfolio management.

This documentation provides a comprehensive guide for developers, covering the system's architecture, API, and setup procedures.

### Documentation Sections

To get started, explore the following sections:

- [**Installation Guide**](./installation.md "null"): A step-by-step guide to setting up the backend on a local machine for development and testing.
    
- [**API Reference**](./reference.md "null"): Detailed documentation for every API endpoint, including request/response schemas and examples.
    
- **Architecture Deep Dive**:
    
    - [**Overview**](./architecture/overview.md "null"): A high-level look at the system's design, components, and core principles.
        
    - [**Database Schema**](./architecture/database-schema.md "null"): An explanation of the Prisma data models and their relationships.
        
    - [**Provider Pattern**](./architecture/providers.md "null"): A detailed look at the provider pattern, the cornerstone of the backend's modularity.
        
    - [**Authentication Flow**](./architecture/auth.md "null"): A sequence diagram and explanation of the user authentication and request resolution process.
        
    - [**Request Flow**](./architecture/request-flow.md "null"): An explanation of how a typical request is processed after authentication.
        

## About the NextNonce Backend

The NextNonce backend is a modern, robust, and scalable application designed to provide a seamless and powerful portfolio tracking experience.

### Key Features

- **Multi-Chain Support**: Tracks assets across 15 of the most popular EVM blockchains, including Ethereum, Polygon, Base, Arbitrum, and more.
    
- **Comprehensive Wallet Support**: Fetches balances for both standard **Simple (EOA)** and more complex **Smart Contract** wallets.
    
- **Unified Token View**: A standout feature that consolidates popular tokens (like ETH, USDC, USDT) from across all supported networks. It presents an aggregated total balance for each unified token, while also allowing users to see the specific balance on each individual chain.
    
- **Robust Authentication**: All API endpoints are secured. Requests must include a valid JWT issued for the [NextNonce mobile application](https://github.com/NextNonce/Mobile "null").
    
- **Provider-Based Architecture**: Core functionalities like authentication, database interaction, balance fetching, and caching are abstracted behind interfaces, making the system modular and easy to extend with new providers.
    
- **Advanced Caching**: Implements sophisticated caching strategies using Redis to ensure high performance and low latency for frequently accessed data.
    
- **Distributed Rate Limiting**: Protects the application and its underlying data providers from abuse by using a Redis-backed system to synchronize rate limits across all running backend instances.
    
- **Detailed Logging & Tracing**: Integrated with New Relic for performance monitoring and structured, file-based logging for easier debugging.
    

### Technology Stack

- **Framework**: [NestJS](https://nestjs.com/ "null") (v11)
    
- **Language**: [TypeScript](https://www.typescriptlang.org/ "null") (v5)
    
- **Database**: [PostgreSQL](https://www.postgresql.org/ "null") managed via [Supabase](https://supabase.com/ "null")
    
- **ORM**: [Prisma](https://www.prisma.io/ "null")
    
- **Authentication**: [Supabase Auth](https://supabase.com/auth "null"), with JWTs handled by `passport-jwt`.
    
- **Caching**: [Redis](https://redis.io/ "null") via `ioredis`.
    
- **API Specification**: [OpenAPI (Swagger)](https://swagger.io/ "null")
    
- **Testing**: [Jest](https://jestjs.io/ "null") for both unit and end-to-end (e2e) tests.
    
- **Containerization**: [Docker](https://www.docker.com/ "null") & Docker Compose.
    

### High-Level Architecture

The backend is designed as a modular, service-oriented application following the principles of clean architecture.

- **Modularity**: The application is divided into feature-specific modules (e.g., `User`, `Portfolio`, `Wallet`, `Balance`, `Token`). Each module encapsulates its own logic, controllers, services, and DTOs. This separation of concerns makes the codebase easier to maintain, test, and scale.
    
- **Provider Abstraction**: A core architectural pattern is the use of provider interfaces. This design allows for swapping out providers (e.g., moving from Supabase to another auth service, or adding a new balance data source) with minimal changes to the core application logic.
    
- **Database & ORM**: Prisma serves as the interface to the PostgreSQL database. The schema is defined in `prisma/schema.prisma`, and migrations are managed by the Prisma CLI.
    
- **Request Lifecycle**: An incoming request is first validated by an authentication guard. A custom interceptor then resolves the authenticated user to an internal database record. The request is routed to a controller, which delegates to a service where the core business logic is executed. A global exception filter catches any unhandled errors and formats them into a standardized response.