# Product API Sequence Diagrams

This document contains sequence diagrams showing the request flow for different scenarios in the Product Catalog API.

## 1. Successful Product Retrieval (HTTP 200)

```mermaid
sequenceDiagram
    actor Client
    participant Express as Express Server<br/>(index.js)
    participant Router as Product Router<br/>(routes/product.js)
    participant Service as Product Service<br/>(service/product.js)

    Client->>Express: GET /product/1
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Router: Route to /product/:id
    Router->>Router: Extract id from params
    Router->>Service: validateProductId("1")
    Service->>Service: Check regex /^[0-9]+$/
    Service-->>Router: true (valid)
    Router->>Service: getProductById("1")
    Service->>Service: Check if id === "3"
    Service->>Service: Lookup products[1]
    Service-->>Router: Product object
    Router->>Router: product exists
    Router-->>Express: 200 JSON response
    Express-->>Client: 200 OK<br/>{"id": 1, "product_name": "Product 01", "price": 100.50}
```

## 2. Product Not Found (HTTP 404)

```mermaid
sequenceDiagram
    actor Client
    participant Express as Express Server<br/>(index.js)
    participant Router as Product Router<br/>(routes/product.js)
    participant Service as Product Service<br/>(service/product.js)

    Client->>Express: GET /product/2
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Router: Route to /product/:id
    Router->>Router: Extract id from params
    Router->>Service: validateProductId("2")
    Service->>Service: Check regex /^[0-9]+$/
    Service-->>Router: true (valid)
    Router->>Service: getProductById("2")
    Service->>Service: Check if id === "3"
    Service->>Service: Lookup products[2]
    Service-->>Router: null (not found)
    Router->>Router: product is null
    Router-->>Express: 404 JSON response
    Express-->>Client: 404 Not Found<br/>{"message": "Product id=2 not found in system"}
```

## 3. System Error (HTTP 500)

```mermaid
sequenceDiagram
    actor Client
    participant Express as Express Server<br/>(index.js)
    participant Router as Product Router<br/>(routes/product.js)
    participant Service as Product Service<br/>(service/product.js)

    Client->>Express: GET /product/3
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Router: Route to /product/:id
    Router->>Router: Extract id from params
    Router->>Service: validateProductId("3")
    Service->>Service: Check regex /^[0-9]+$/
    Service-->>Router: true (valid)
    Router->>Service: getProductById("3")
    Service->>Service: Check if id === "3"
    Service->>Service: throw Error("System Error")
    Service--xRouter: Error thrown
    Router->>Router: catch error
    Router-->>Express: 500 JSON response
    Express-->>Client: 500 Internal Server Error<br/>{"message": "System Error"}
```

## 4. Invalid Input - Validation Error (HTTP 400)

```mermaid
sequenceDiagram
    actor Client
    participant Express as Express Server<br/>(index.js)
    participant Router as Product Router<br/>(routes/product.js)
    participant Service as Product Service<br/>(service/product.js)

    Client->>Express: GET /product/abc
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Router: Route to /product/:id
    Router->>Router: Extract id from params
    Router->>Service: validateProductId("abc")
    Service->>Service: Check regex /^[0-9]+$/
    Service-->>Router: false (invalid)
    Router->>Router: Validation failed
    Router-->>Express: 400 JSON response
    Express-->>Client: 400 Bad Request<br/>{"message": "Invalid product ID. Must be numeric only."}
```

## 5. Security - SQL Injection Attempt (HTTP 400)

```mermaid
sequenceDiagram
    actor Attacker
    participant Express as Express Server<br/>(index.js)
    participant Router as Product Router<br/>(routes/product.js)
    participant Service as Product Service<br/>(service/product.js)

    Attacker->>Express: GET /product/1' OR '1'='1
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Router: Route to /product/:id
    Router->>Router: Extract id from params
    Router->>Service: validateProductId("1' OR '1'='1")
    Service->>Service: Check regex /^[0-9]+$/
    Service-->>Router: false (invalid)
    Router->>Router: Validation failed
    Router-->>Express: 400 JSON response
    Express-->>Attacker: 400 Bad Request<br/>{"message": "Invalid product ID. Must be numeric only."}
    Note over Service: Attack prevented!<br/>Service layer never reached
```

## 6. Health Check Endpoint

```mermaid
sequenceDiagram
    actor Client
    participant Express as Express Server<br/>(index.js)

    Client->>Express: GET /health
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Express: Health check handler
    Express-->>Client: 200 OK<br/>{"status": "OK"}
```

## 7. Unknown Endpoint (HTTP 404)

```mermaid
sequenceDiagram
    actor Client
    participant Express as Express Server<br/>(index.js)
    participant Router as Product Router<br/>(routes/product.js)

    Client->>Express: GET /unknown
    Express->>Express: Apply middleware<br/>(JSON parsing)
    Express->>Express: Check health endpoint
    Express->>Router: Check product routes
    Router-->>Express: No match
    Express->>Express: 404 handler (middleware)
    Express-->>Client: 404 Not Found<br/>{"message": "Endpoint not found"}
```

## Architecture Flow Summary

### Request Processing Pipeline

```mermaid
flowchart TD
    A[Client Request] --> B[Express Server<br/>index.js]
    B --> C{Route Match?}
    C -->|/health| D[Health Check Handler]
    C -->|/product/:id| E[Product Router<br/>routes/product.js]
    C -->|Unknown| F[404 Handler]

    E --> G{Validate ID}
    G -->|Invalid| H[400 Bad Request]
    G -->|Valid| I[Product Service<br/>service/product.js]

    I --> J{ID === '3'?}
    J -->|Yes| K[Throw Error]
    J -->|No| L{Product Exists?}

    K --> M[500 System Error]
    L -->|Yes| N[200 Success]
    L -->|No| O[404 Not Found]

    D --> P[Response to Client]
    F --> P
    H --> P
    M --> P
    N --> P
    O --> P

    P --> Q[Client Receives Response]
```

### Component Interaction

```mermaid
flowchart LR
    A[Client] <-->|HTTP| B[Express Server]
    B <-->|Mount Routes| C[Product Router]
    C <-->|Business Logic| D[Product Service]
    D <-->|Data Access| E[(Mock Data Store)]

    style B fill:#e1f5ff
    style C fill:#fff4e1
    style D fill:#e8f5e9
    style E fill:#fce4ec
```

## Testing Flow

```mermaid
sequenceDiagram
    actor Tester
    participant Jest as Jest Test Runner
    participant Supertest as Supertest
    participant App as Express App<br/>(index.js)
    participant Router as Product Router
    participant Service as Product Service

    Tester->>Jest: npm test
    Jest->>Jest: Load test suite<br/>(tests/product.test.js)
    Jest->>Supertest: Initialize test request
    Supertest->>App: GET /product/1<br/>(NODE_ENV=test)
    Note over App: Server doesn't listen<br/>in test mode
    App->>Router: Route request
    Router->>Service: Process request
    Service-->>Router: Return result
    Router-->>App: Format response
    App-->>Supertest: HTTP response
    Supertest-->>Jest: Assertion result
    Jest->>Jest: Verify expectations
    Jest-->>Tester: Test results<br/>(15 passed)
```

## Error Handling Flow

```mermaid
flowchart TD
    A[Request Received] --> B{Validation Pass?}
    B -->|No| C[Return 400<br/>Validation Error]
    B -->|Yes| D{Try-Catch Block}

    D -->|Exception Thrown| E{Error Type?}
    E -->|System Error id=3| F[Return 500<br/>System Error]
    E -->|Other Error| G[Global Error Handler<br/>Return 500]

    D -->|No Exception| H{Product Found?}
    H -->|Yes| I[Return 200<br/>Product Data]
    H -->|No| J[Return 404<br/>Not Found]

    C --> K[Response Sent]
    F --> K
    G --> K
    I --> K
    J --> K
```
