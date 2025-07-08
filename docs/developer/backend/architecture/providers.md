# Providers

A cornerstone of the **NextNonce** backend architecture is the consistent and rigorous use of the **Provider Pattern**. This design principle is fundamental to creating a system that is modular, highly testable, and adaptable to future changes. By programming to interfaces rather than concrete implementations, the application avoids tight coupling between components and external services, ensuring long-term maintainability and flexibility.

The core idea is simple yet powerful: for any external-facing or complex logic (like authentication, data fetching, or caching), an abstraction layer (an interface) is defined. This interface dictates a contract — a set of methods that any concrete implementation must adhere to. The rest of the application interacts only with this interface, remaining completely unaware of the underlying details of the specific implementation being used.

This approach yields several critical advantages:

- **Interchangeability**: If a decision is made to switch from one external data source to another (e.g., from Dune to a different analytics platform), only a new provider class needs to be created. The service consuming the data remains unchanged.
    
- **Testability**: When writing unit tests for services, it's trivial to supply a mock implementation of a provider interface. This allows for testing the service's logic in complete isolation, without making actual network calls or database queries.
    
- **Separation of Concerns**: Each provider is responsible for a single task. This keeps the core business logic in services clean and free from vendor-specific details.
    

A core architectural pattern is the use of provider interfaces. For instance:

- `AuthProvider` (`/src/auth/interfaces/auth-provider.interface.ts`) abstracts the authentication logic, with `SupabaseAuthProvider` being the concrete implementation.
    
- `BalanceProvider` (`/src/balance/interfaces/balance-provider.interface.ts`) defines the contract for fetching wallet balances, currently implemented by `OkxDexBalanceProvider`.
    
- `CacheProvider` (`/src/cache/interfaces/cache-provider.interface.ts`) abstracts caching operations, with `RedisCacheProvider` providing the implementation.
    

## Deep Dive: The `MetadataModule` Example

To understand how this pattern works in practice, let's examine the `MetadataModule`. Its responsibility is to fetch metadata (like name, symbol, and decimals) for tokens a user holds.

The module follows a standard structure:

```shell
src/metadata
├── dto/                      # Data Transfer Objects for this module
│   └── external-token-metadata.dto.ts
├── interfaces/               # The abstraction (the contract)
│   └── token-metadata-provider.interface.ts
├── providers/                # The concrete implementation
│   └── dune-token-metadata.provider.ts
├── metadata.module.ts        # Wires everything together
└── metadata.service.ts       # Consumes the provider and orchestrates logic
```

### 1. The Interface (`token-metadata-provider.interface.ts`)

This file defines the contract. Any class that provides token metadata _must_ adhere to this `TokenMetadataProvider` interface.

```ts
// src/metadata/interfaces/token-metadata-provider.interface.ts

import { ExternalTokenMetadataDto } from '@/metadata/dto/external-token-metadata.dto';

// Use a Symbol as the injection token to avoid name collisions.
// This is a robust practice in NestJS for non-class providers.
export const TOKEN_METADATA_PROVIDER = Symbol('TOKEN_METADATA_PROVIDER');

export interface TokenMetadataProvider {
    /**
     * Given a wallet address and an array of chain names,
     * return metadata for tokens that the wallet owns on those chains.
     */
    getByWalletAddress(
        walletAddress: string,
        chainNames: string[],
    ): Promise<ExternalTokenMetadataDto[] | undefined>;
}
```

The key element here is `TOKEN_METADATA_PROVIDER`. Since an interface doesn't exist at runtime in JavaScript, we use a `Symbol` as a unique, collision-proof token for NestJS's dependency injection system.

### 2. The Concrete Implementation (`dune-token-metadata.provider.ts`)

This class is the concrete implementation of the `TokenMetadataProvider` interface. It knows how to communicate specifically with the **Dune Sim API**.

Its responsibilities include:

- Implementing the `getByWalletAddress` method.
    
- Handling Dune-specific API details, like the base URL, authentication headers (`X-Sim-Api-Key`), and rate limiting.
    
- Mapping the response from the Dune API into the application's standardized `ExternalTokenMetadataDto`.
    

This class contains all the messy, vendor-specific logic, keeping it isolated from the rest of the application.

### 3. Wiring It All Together (`metadata.module.ts`)

The `MetadataModule` is where the magic happens. It tells the NestJS dependency injection system how to resolve the `TokenMetadataProvider` interface.

```ts
// src/metadata/metadata.module.ts

@Module({
    imports: [ChainModule],
    providers: [
        // This is the crucial part
        {
            provide: TOKEN_METADATA_PROVIDER, // When this token is requested...
            useClass: DuneTokenMetadataProvider, // ...provide this concrete class.
        },
        MetadataService,
    ],
    exports: [MetadataService],
})
export class MetadataModule {}
```

This configuration explicitly states: "When any part of the application asks for the dependency identified by the `TOKEN_METADATA_PROVIDER` token, instantiate and provide the `DuneTokenMetadataProvider` class."

### 4. Consuming the Provider (`metadata.service.ts`)

The `MetadataService` is the consumer of the provider. It orchestrates the business logic, such as fetching data and caching it. Notice its constructor:

```ts
// src/metadata/metadata.service.ts

@Injectable()
export class MetadataService {
    constructor(
        private readonly databaseService: DatabaseService,
        private readonly cacheService: CacheService,
        private readonly chainService: ChainService,
        @Inject(TOKEN_METADATA_PROVIDER)
        private readonly tokenMetadataProvider: TokenMetadataProvider,
    ) {
        // ...
    }

    async getTokensWithMetadataByWalletAddress(
        walletAddress: string,
        chainNames: string[],
    ): Promise<TokenWithMetadata[]> {
        // ...
        const providedTokensMetadata =
            await this.tokenMetadataProvider.getByWalletAddress(
                walletAddress,
                chainNames,
            );
        // ...
    }
}
```

The most important takeaway is that **`MetadataService` has no idea that `DuneTokenMetadataProvider` exists**. It only depends on the `TokenMetadataProvider` interface. It knows it can call `getByWalletAddress`, but it doesn't know or care _how_ that method is implemented.

This makes the `MetadataService` incredibly robust and flexible. If we decide to switch from Dune to another provider, we would simply:

1. Create a new provider class, e.g., `AlchemyTokenMetadataProvider`.
    
2. Update a single line in `metadata.module.ts` to `useClass: AlchemyTokenMetadataProvider`.
    

No code would need to be changed in `MetadataService` itself. This pattern is the foundation of the backend's clean, decoupled, and scalable architecture.