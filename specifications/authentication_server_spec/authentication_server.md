# 🔒 Authentication_Server Specification

## Overview
Standard Oauth 2.0 server according to [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749).
- Client Credentials Flow

## 💾 Auth Storage model
```mermaid
---
title: Oauth Server ERD
---
%%{init: {'theme':'base'}}%%

erDiagram
    CLIENTS {
        UUID id PK "Primary Key"
        VARCHAR(255) client_id "Public client identifier"
        VARCHAR(255) client_secret "Client's secret"
        TEXT redirect_uri "Allowed redirect URIs"
        TEXT scopes "Scopes associated with the client"
        TIMESTAMP created_at "Record creation time"
    }

    TOKENS {
        UUID id PK "Primary Key"
        TEXT token "Issued access token"
        UUID client_id FK "Foreign Key -> CLIENTS.id"
        TEXT scope "Scopes associated with the token"
        TIMESTAMP expires_at "Token expiration time"
        TIMESTAMP created_at "Token creation time"
    }

    USERS {
        UUID id PK "Primary Key"
        VARCHAR(255) username "Unique username"
        TEXT password_hash "Hashed password for the user"
        TIMESTAMP created_at "Record creation time"
    }

    TOKEN_INTROSPECTION_LOGS {
        UUID id PK "Primary Key"
        TEXT token "Introspected token"
        UUID client_id FK "Foreign Key -> CLIENTS.id"
        JSONB result "Introspection result (e.g., active)"
        TIMESTAMP timestamp "Time of introspection"
    }

    %% Relationships
    CLIENTS ||--o{ TOKENS : "issues"
    TOKENS ||--o{ TOKEN_INTROSPECTION_LOGS : "are introspected"
```

## 🔒 Auth API
**Standard Oauth 2.0 Client Credentials Flow.**

#### 🎯 Purpose
The `Authorization Server` is designed to facilitate secure authorization and authentication in applications using the OAuth 2.0 protocol. It serves as the central authority for issuing, validating, and managing access tokens, which allow clients to securely access protected resources.

##### Key Objectives

1. **Secure Authorization**:
   - Allows clients (e.g., applications, services) to authenticate themselves and obtain access tokens using the OAuth 2.0 **Client Credentials Flow**.
   - Ensures secure exchange of sensitive credentials like `client_id` and `client_secret`.

2. **Token Introspection**:
   - Provides a mechanism to validate and retrieve metadata for tokens through the `/oauth2/introspect` endpoint.
   - Helps resource servers verify the validity and scope of tokens before granting access to protected resources.

3. **Centralized Token Management**:
   - Enables the issuance, storage, and management of tokens, ensuring they are securely generated, tied to specific clients, and have appropriate lifetimes and permissions.

4. **Scalability and Interoperability**:
   - Designed to work across multiple clients and resource servers in a distributed architecture.
   - Supports standards-based communication to ensure compatibility with various OAuth 2.0-compliant systems.

For more details see: [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)

### 🧠 Business Logic

### 📐 API Structure
- [**Authentication Server OpenAPI**](authentication_server-openapi.yaml)

#### ⚙️ POST /oatuh2/authorize
```mermaid
---
title: Simplified SingUp for the DEMO
---
%%{init: {'theme':'base'}}%%
sequenceDiagram
    participant Client as OAuth Client
    participant AuthServer as Authorization Server
    participant DB as Database

    %% /oauth2/authorize Flow
    Client->>+AuthServer: POST /oauth2/authorize with client_id, client_secret, grant_type
    AuthServer->>+DB: Validate client_id and client_secret
    DB-->>-AuthServer: Client validation result

    opt IF User by client_id and client_secret does NOT exist
        AuthServer->>AuthServer: Create new User with client_id and client_secret
    end

    AuthServer->>AuthServer: Check grant_type == "client_credentials"
    AuthServer->>AuthServer: Generate access token (JWT)
    AuthServer->>+DB: Store access token with metadata (scope, expires_at)
    deactivate DB
    AuthServer-->>-Client: 200 OK with access_token, token_type, expires_in
```

```mermaid
---
title: Standard Oauth 2.0 Client Credentials Flow - Introspect
---
sequenceDiagram
    participant Client as OAuth Client
    participant AuthServer as Authorization Server
    participant DB as Database

    %% /oauth2/authorize Flow
    Client->>AuthServer: POST /oauth2/authorize with client_id, client_secret, grant_type
    AuthServer->>DB: Validate client_id and client_secret
    DB-->>AuthServer: Client validation result
    AuthServer->>AuthServer: Check grant_type == "client_credentials"
    AuthServer->>AuthServer: Generate access token (JWT)
    AuthServer->>DB: Store access token with metadata (scope, expires_at)
    AuthServer-->>Client: 200 OK with access_token, token_type, expires_in
```

#### ⚙️ POST /oatuh2/introspect
```mermaid
---
title: Standard Oauth 2.0 Client Credentials Flow - Introspect
---
sequenceDiagram
    participant Client as OAuth Client
    participant AuthServer as Authorization Server
    participant DB as Database

    %% /oauth2/introspect Flow
    Client->>AuthServer: POST /oauth2/introspect with token
    AuthServer->>DB: Lookup token in database
    DB-->>AuthServer: Token metadata (if valid)
    AuthServer->>AuthServer: Validate token expiration and scope
    alt Token is valid
        AuthServer-->>Client: 200 OK with token details (active: true, scope, exp, etc.)
    else Token is invalid
        AuthServer-->>Client: 200 OK with { active: false }
    end
```

## 📑 Related Documentation
- [🏗️ Solution Architecture - Component Diagram](../../solution_design/solution_architecture/assets/quiz_backend-component_diagram-simplified.svg)