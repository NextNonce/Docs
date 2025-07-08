# Authentication

Every authenticated request to the **NextNonce** API undergoes a rigorous, multi-step process to validate the caller's identity and attach a secure, internal user context to the request. This flow is designed for security, performance, and modularity, leveraging guards, interceptors, caching, and a provider-based architecture.

This document details the complete lifecycle of an authenticated request, from the moment it leaves the client to when it is processed by a controller.

### Security & Design Philosophy: Internal IDs

A core security principle in the **NextNonce** backend is the **abstraction and protection of internal entity identifiers**. The primary keys for models like `User`, `Wallet`, `Token`, etc., are UUIDs generated and managed internally. These internal IDs are **never** exposed to the client-side application (except for portfolio's id). This prevents enumeration attacks and decouples the client's data representation from the backend's database structure, providing a more secure and flexible API. The entire flow described below is designed to securely resolve a client's request to an internal `User` entity without ever revealing its ID.

### Step-by-Step Request Lifecycle

The process can be broken down into two major phases: **Authentication** (validating the JWT and identifying the external auth user) and **User Resolution** (finding the corresponding internal database user).


1. **Client → Cloudflare**
    
    - The client issues an HTTPS request to https://api.nextnonce.com/v1/....
        
    - Cloudflare sits in front of the API, blocking bad actors (DDoS, IP blacklists, etc.). Only “good” traffic ever reaches backend.
        
    
2. **Cloudflare → NestJS HTTP**
    
    - Validated requests are forwarded to the NestJS server (Express under the hood), as defined in main.ts.
        
    
3. **NestJS → JwtAuthGuard**
    
    - Before hitting any controller, the JwtAuthGuard (registered globally or per-route) intercepts.
        
    - Under the hood it uses the JwtStrategy.validate() method.
        
    
4. **JwtStrategy.validate(token)**
    
    - **No token present** → throws NotFoundException.
        
        - Caught by AllExceptionsFilter, transformed into a 401 Unauthorized JSON response.
            
        
    - **Token present** → returns the JWT payload (with sub, exp, etc.).
        
    
5. **Back in JwtAuthGuard**
    
    - Calls authService.getAuthUserByToken(token) to resolve the full AuthUserDto.
        
    
6. **AuthService → CacheService**
    
    - CacheService.get&lt;TokenAuthDto&gt;(key = token) checks the Redis (via CacheProvider).
        
    - **Cache hit** → instant return of AuthUserDto.
        
    - **Cache miss** → falls through to the AuthProvider.
        
    
7. **AuthService → AuthProvider.getAuthUserByJwt(token)**
    
    - The default is SupabaseAuthProvider. It calls Supabase’s user-lookup API.
        
    - **No user returned** → throws NotFoundException → 401 via AllExceptionsFilter.
        
    - **User returned** → mapped into AuthUserDto.
        
    
8. **AuthService caches & returns**
    
    - Stores the new AuthUserDto under the token key in Redis.
        
    - Returns AuthUserDto to the guard.
        
    
9. **JwtAuthGuard final checks**
    

    ```ts
    if (authUser.id !== payload.sub) {
        throw new UnauthorizedException('Invalid token');
    }
    ```

    - **Mismatch** → 401 Unauthorized.
        
    - **Match** → call next().
        
    
10. **UserInterceptor**
    
    - Now that the guard has passed, UserInterceptor runs.
        
    - It calls userService.findByAuthUser(authUser) to map the external AuthUserDto → internal User record.
        
    
11. **UserService.findByAuthUser(authUser)**
    
    - **Cache lookup** by cacheService.get&lt;User&gt;(key = authUser.id).
        
        - Hit → return cached User.
            
        - Miss → query the database via Prisma:
            
        
    

    ```ts
    const user = await this.databaseService.user.findFirst({
        where: { auth: { providerUid: authUser.id } }
    });
    ``` 
            
    - **No user** → log error, throw NotFoundException('User not found') → 404 via filter.
        
    - **User found** → cache it and return the User entity.
            
        
    
12. **Controller receives @CurrentUser()**
    
    - UserInterceptor attaches the internal User object to the request context.
        
    - Controller handlers now have a typed, sanitized user, never exposing raw database IDs to the client.
        
    

---


![](../../../images/auth.svg)