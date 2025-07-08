# Instalation Guide

This guide provides comprehensive, step-by-step instructions for setting up and running the NextNonce backend on your local machine for development and testing purposes.

## Part 1: System Requirements

Before you begin, ensure your development environment meets the following minimum requirements.

### 1.1. Hardware Specifications

While the application can run on less, these are the recommended specs for a smooth development experience.

- **CPU:** 2+ Cores
    
- **RAM:** 4 GB or more (Node.js, Prisma, and running multiple services can be memory-intensive)
    
- **Disk Space:** 5 GB of free space (for the project, dependencies, Docker images)
    
- **Operating System:** macOS, Windows (with WSL2), or any major Linux distribution.
    

### 1.2. Software Prerequisites

You must install the following software.

- **Git:** To clone the repository. [Download Git](https://git-scm.com/downloads "null")
    
- **Node.js:** The runtime environment. Use version `22.x` as specified in the `Dockerfile`. [Download Node.js](https://nodejs.org/en/ "null") or use a version manager like [nvm](https://github.com/nvm-sh/nvm "null").
    
- **Yarn:** The package manager for the project. [Install Yarn](https://yarnpkg.com/getting-started/install "null")
    
- **Docker & Docker Compose:** For containerized deployment. [Download Docker Desktop](https://www.docker.com/products/docker-desktop/ "null")
    

## Part 2: External Services Setup

The backend relies on several external services for its core functionality. You will need to create accounts and obtain API keys/credentials for each.

1. **Supabase (Database & Auth):**
    
    - Go to [Supabase](https://supabase.com/ "null") and create a new project.
        
    - You will need:
        
        - **Database Connection String:** Go to Settings > Database > Connection string > URI in your project.
            
        - **Project API Keys:** Go to `Settings` > `API`. You need the `Project URL`, `anon` key, and `service_role` key.
            
        - **JWT Secret:** Go to `Settings` > `API` > `JWT Settings`.
            
2. **Redis (Caching & Rate Limiting):**
    
    - You can run Redis locally via Docker or use a cloud provider like [Redis Labs](https://redis.com/try-free/ "null") or [Upstash](https://upstash.com/ "null").
        
    - You will need the `host`, `port`, and `password` for your Redis instance.
        
3. **Alchemy (Blockchain Node Provider):**
    
    - Go to [Alchemy](https://www.alchemy.com/ "null"), create an account, and set up a new app.
        
    - You will need the **API Key** for your app.
        
4. **Dune (On-Chain Data Analytics):**
    
    - Go to [Sim Dune](https://sim.dune.com/), create an account, and generate an API key from your settings.
        
    - You will need the **API Key**.
        
5. **OKX (Balance & Metadata Provider):**
    
    - Go to the [OKX Web3 Developer Portal](https://web3.okx.com/build/dev-portal/project "null") and create a project.
        
    - You will need the `API Key`, `Secret Key`, and `Passphrase`.
        
6. **New Relic (Optional Performance Monitoring):**
    
    - If you plan to use monitoring, go to [New Relic](https://newrelic.com/ "null"), create an account, and get your `License Key` and an `App Name`.
        

## Part 3: Local Installation & Configuration

Now, let's clone the code and configure the application.

### Step 1: Clone the Repository

Open your terminal and clone the repository to your local machine.

```
git clone https://github.com/NextNonce/Backend.git
cd backend
```

### Step 2: Install Dependencies

Use Yarn to install all the project's dependencies as defined in `yarn.lock`.

```
yarn install --frozen-lockfile
```

### Step 3: Configure Environment Variables

The application uses a `.env` file to manage all secrets and environment-specific configurations.

1. Create a `.env` file by copying the example file:
    
    ```
    cp .env.example .env
    ```
    
2. Open the newly created `.env` file in a text editor and populate it with the credentials you gathered in **Part 2**.
    
    ```
    # Set to "development" for local work
    NODE_ENV=development
    
    # Can be left empty for local development
    MODE=
    
    # Prisma / Supabase
    DATABASE_URL="postgresql://postgres:[YOUR-PASSWORD]@[YOUR-DB-HOST]:5432/postgres"
    DIRECT_URL="postgresql://postgres:[YOUR-PASSWORD]@[YOUR-DB-HOST]:5432/postgres"
    
    # Supabase Auth
    SUPABASE_URL=https://[YOUR-PROJECT-ID].supabase.co
    SUPABASE_KEY=[YOUR-SUPABASE-SERVICE-ROLE-KEY]
    SUPABASE_ANON_KEY=[YOUR-SUPABASE-ANON-KEY]
    JWT_SECRET=[YOUR-SUPABASE-JWT-SECRET]
    
    # Cache
    CACHE_PREFIX=nextnonce
    CACHE_VERSION=v1
    
    # Redis
    REDIS_HOST=localhost # or your remote Redis host
    REDIS_PORT=6379
    REDIS_PASSWORD= # your redis password, if any
    
    # API Keys
    ALCHEMY_API_KEY=[YOUR-ALCHEMY-API-KEY]
    DUNE_API_KEY=[YOUR-DUNE-API-KEY]
    OKX_DEV_API_KEY=[YOUR-OKX-API-KEY]
    OKX_DEV_SECRET_KEY=[YOUR-OKX-SECRET-KEY]
    OKX_DEV_PASSPHRASE=[YOUR-OKX-PASSPHRASE]
    
    # New Relic (Optional)
    NEW_RELIC_APP_NAME=NextNonce-Backend-Dev
    NEW_RELIC_LICENSE_KEY=
    ```
    

### Step 4: Set Up the Database

Run the Prisma migration command to set up your database schema. This will read your `prisma/schema.prisma` file and apply the necessary table structures to the database specified in your `DATABASE_URL`.

```
npx prisma migrate dev
```

This will also generate the Prisma Client based on your schema, which is necessary for the application to communicate with the database.

## Part 4: Building and Running the Application

You can run the application in several ways.

### Method A: Local Development Mode (Recommended)

This method uses the NestJS CLI to run the app with hot-reloading, which automatically restarts the server when you save a file. This is ideal for active development.

```
yarn start:dev
```

The server will start, and you should see output in your terminal indicating it's running.

### Method B: Local Production Mode

This method simulates how the app would run in production. It first builds the JavaScript files and then runs them directly with Node.js.

1. **Build the application:**
    
    ```
    yarn build
    ```
    
    This compiles the TypeScript code from `src/` into JavaScript in the `dist/` directory.
    
2. **Run the built application:**
    
    ```
    yarn start:prod
    ```
    

### Method C: Using Docker Compose

This method uses Docker to create a containerized environment for the application, which is excellent for ensuring consistency across different machines.

1. Ensure Docker Desktop is running.
    
2. Run the following command from the project root:
    
    ```
    docker-compose up --build -d
    ```
    
    This command will:
    
    - Build the Docker image for the backend as defined in the `Dockerfile`.
        
    - Start a container for the backend application.
        

## Part 5: Verifying the Installation

Once the application is running (using any of the methods above), you can verify that it's working correctly.

1. **Swagger API Documentation**: The application serves interactive API documentation. Access it here: `http://localhost:3000/v1/docs`
    
    You should see the Swagger UI, which lists all available API endpoints. This is the best way to confirm that the application has started successfully and all controllers have been loaded.
    

## Part 6: Running Tests

Run the included test suites to ensure all modules are functioning as expected.

1. **Run all unit tests:**
    
    ```
    yarn test
    ```
    
2. **Run end-to-end (e2e) tests:** _Note: E2E tests may require a separate test database and a running application instance. Ensure your `.env` is configured correctly for the test environment._
    
    ```
    yarn test:e2e
    ```
    

You have successfully set up, configured, and run the NextNonce backend on your local machine. Happy coding!