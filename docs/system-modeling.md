# System Modeling

This document provides comprehensive system modeling diagrams for the Mercado Libre Payment API, including data models, architecture, and process flows.

---

## Table of Contents

- [Data Models (ERD)](#data-models-erd)
- [System Architecture](#system-architecture)
- [Authentication Flow](#authentication-flow)
- [CRUD Flow](#crud-flow)
- [Security Flow](#security-flow)
- [Payment Processing Flow](#payment-processing-flow)
- [Component Diagram](#component-diagram)
- [Deployment Diagram](#deployment-diagram)

---

## Data Models (ERD)

The following Entity Relationship Diagram represents the data models used in the payment system:

```mermaid
erDiagram
    PAYMENT ||--o{ PAYMENT_METHOD : "uses"
    PAYMENT ||--|| PAYER : "belongs to"
    PAYMENT ||--o{ TRANSACTION : "generates"
    
    PAYMENT {
        string id PK
        decimal transaction_amount
        string description
        string status
        string status_detail
        datetime date_created
        datetime date_approved
        string payment_method_id
    }
    
    PAYMENT_METHOD {
        string type PK
        string card_number
        string expiration_month
        string expiration_year
        string security_code
        int installments
    }
    
    PAYER {
        string email PK
        string identification_type
        string identification_number
        string first_name
        string last_name
        string zip_code
        string street_name
        string street_number
        string neighborhood
        string city
        string federal_unit
    }
    
    TRANSACTION {
        string id PK
        string payment_id FK
        string external_resource_url
        string qr_code
        string qr_code_base64
        string ticket_url
    }
```

### Data Model Descriptions

#### Payment Entity

Core entity representing a payment transaction.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique payment identifier from Mercado Pago |
| `transaction_amount` | decimal | Payment amount in BRL |
| `description` | string | Payment description/purpose |
| `status` | string | Current payment status |
| `status_detail` | string | Detailed status information |
| `date_created` | datetime | Payment creation timestamp |
| `date_approved` | datetime | Payment approval timestamp |
| `payment_method_id` | string | Payment method identifier |

#### Payment Method Entity

Stores payment method-specific information.

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Payment method type (card, pix, boleto) |
| `card_number` | string | Credit card number (tokenized) |
| `expiration_month` | string | Card expiration month |
| `expiration_year` | string | Card expiration year |
| `security_code` | string | CVV/CVC code |
| `installments` | int | Number of installments |

#### Payer Entity

Contains payer information.

| Field | Type | Description |
|-------|------|-------------|
| `email` | string | Payer's email address |
| `identification_type` | string | ID type (CPF, CNPJ) |
| `identification_number` | string | Tax ID number |
| `first_name` | string | First name (boleto only) |
| `last_name` | string | Last name (boleto only) |
| `zip_code` | string | Postal code (boleto only) |
| `street_name` | string | Street name (boleto only) |
| `street_number` | string | Street number (boleto only) |
| `neighborhood` | string | Neighborhood (boleto only) |
| `city` | string | City (boleto only) |
| `federal_unit` | string | State abbreviation (boleto only) |

#### Transaction Entity

Stores transaction-specific details.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Transaction identifier |
| `payment_id` | string | Reference to payment |
| `external_resource_url` | string | External payment URL (boleto) |
| `qr_code` | string | PIX QR code string |
| `qr_code_base64` | string | PIX QR code as base64 image |
| `ticket_url` | string | Payment ticket URL |

---

## System Architecture

High-level architecture diagram showing system components and their interactions:

```mermaid
graph TB
    subgraph Client["Client Layer"]
        Browser[Web Browser]
        MobileApp[Mobile Application]
        APIConsumer[API Consumer]
    end
    
    subgraph API["API Layer - FastAPI"]
        Routes[API Routes]
        Middleware[Middleware]
        Validation[Request Validation]
        Templates[Jinja2 Templates]
    end
    
    subgraph Services["Service Layer"]
        MPService[MercadoPago Service]
        TokenService[Card Token Service]
        PaymentService[Payment Service]
    end
    
    subgraph External["External Services"]
        MPAPI[Mercado Pago API]
    end
    
    subgraph Storage["Configuration"]
        EnvVars[Environment Variables]
        Secrets[Access Tokens]
    end
    
    Browser -->|HTTP/HTTPS| Routes
    MobileApp -->|HTTP/HTTPS| Routes
    APIConsumer -->|HTTP/HTTPS| Routes
    
    Routes --> Middleware
    Routes --> Validation
    Routes --> Templates
    
    Middleware --> MPService
    Validation --> MPService
    
    MPService --> TokenService
    MPService --> PaymentService
    
    TokenService -->|POST /v1/card_tokens| MPAPI
    PaymentService -->|POST /v1/payments| MPAPI
    
    MPService --> EnvVars
    MPService --> Secrets
    
    style Client fill:#e1f5ff
    style API fill:#fff4e1
    style Services fill:#e8f5e9
    style External fill:#fce4ec
    style Storage fill:#f3e5f5
```

### Architecture Layers

| Layer | Components | Responsibility |
|-------|------------|----------------|
| **Client** | Browser, Mobile App, API Consumer | User interface and API consumption |
| **API** | Routes, Middleware, Validation, Templates | Request handling and response generation |
| **Services** | MercadoPago, Token, Payment Services | Business logic and external integration |
| **External** | Mercado Pago API | Payment processing |
| **Storage** | Environment Variables, Secrets | Configuration management |

---

## Authentication Flow

Authentication flow for API access and Mercado Pago integration:

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant App as Application
    participant Env as Environment
    participant MP as Mercado Pago API
    
    Note over Dev,MP: Setup Phase
    Dev->>Env: Configure MP_ACCESS_TOKEN
    Env->>App: Load token at startup
    
    Note over Dev,MP: Runtime - Card Payment
    App->>App: Read token from environment
    App->>App: Generate X-Idempotency-Key (UUID)
    App->>MP: POST /v1/card_tokens<br/>Headers: Authorization, X-Idempotency-Key
    MP-->>App: Return card token
    
    App->>App: Build payment payload with token
    App->>App: Generate new X-Idempotency-Key
    App->>MP: POST /v1/payments<br/>Headers: Authorization, X-Idempotency-Key
    MP-->>App: Payment response
    
    Note over Dev,MP: Token Management
    App->>Env: Refresh token if expired
    Env-->>App: New access token
```

### Authentication Components

| Component | Purpose | Implementation |
|-----------|---------|----------------|
| **Access Token** | API authentication | Bearer token in Authorization header |
| **Idempotency Key** | Prevent duplicate transactions | UUID per request |
| **Environment Variables** | Secure credential storage | python-decouple |

### Security Headers

```http
Authorization: Bearer APP_USR-xxxxxxxxxxxxxxxx
Content-Type: application/json
X-Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

---

## CRUD Flow

Complete CRUD (Create, Read, Update, Delete) flow for all project operations:

```mermaid
flowchart TD
    subgraph Create["CREATE - Payment Creation"]
        C1[User selects payment method] --> C2[Fill payment details]
        C2 --> C3[Submit payment form]
        C3 --> C4[POST /create_payment]
        C4 --> C5{Payment Method?}
        C5 -->|Card| C6[Tokenize card]
        C5 -->|PIX| C7[Generate PIX]
        C5 -->|Boleto| C8[Generate Boleto]
        C6 --> C9[Create payment in MP]
        C7 --> C9
        C8 --> C9
        C9 --> C10[Return payment result]
    end
    
    subgraph Read["READ - Payment Retrieval"]
        R1[Request payment status] --> R2[GET payment by ID]
        R2 --> R3[Query Mercado Pago API]
        R3 --> R4[Return payment details]
    end
    
    subgraph Update["UPDATE - Payment Status"]
        U1[MP processes payment] --> U2[Status change event]
        U2 --> U3[Webhook notification]
        U3 --> U4[Update local state]
    end
    
    subgraph Delete["DELETE - Payment Cancellation"]
        D1[User requests cancellation] --> D2[Validate payment status]
        D2 --> D3{Can cancel?}
        D3 -->|Yes| D4[Call MP cancellation API]
        D3 -->|No| D5[Return error]
        D4 --> D6[Update payment status]
    end
    
    Create --> Read
    Read --> Update
    Update --> Delete
    
    style Create fill:#e8f5e9
    style Read fill:#e3f2fd
    style Update fill:#fff3e0
    style Delete fill:#ffebee
```

### CRUD Operations Summary

| Operation | Endpoint | Method | Description |
|-----------|----------|--------|-------------|
| **Create** | `/create_payment` | POST | Create new payment |
| **Read** | (via MP API) | GET | Retrieve payment status |
| **Update** | (via Webhook) | POST | Payment status updates |
| **Delete** | (via MP API) | PUT | Cancel payment |

---

## Security Flow

Security flow showing request validation, authentication, and data protection:

```mermaid
flowchart TD
    subgraph Input["Request Input"]
        Req[Incoming Request]
    end
    
    subgraph Validation["Validation Layer"]
        V1{HTTPS?}
        V2[Validate Content-Type]
        V3{Required fields present?}
        V4{Valid payment method?}
        V5{Valid amount > 0?}
        V6{Valid card data?}
    end
    
    subgraph Auth["Authentication Layer"]
        A1[Load access token from env]
        A2{Token valid?}
        A3[Generate idempotency key]
    end
    
    subgraph Processing["Processing Layer"]
        P1[Tokenize sensitive data]
        P2[Encrypt transmission]
        P3[Log non-sensitive data]
    end
    
    subgraph Response["Response Layer"]
        R1{Success?}
        R2[Return payment data]
        R3[Return error message]
        R4[Mask sensitive data]
    end
    
    Req --> V1
    V1 -->|No| R3
    V1 -->|Yes| V2
    V2 --> V3
    V3 -->|No| R3
    V3 -->|Yes| V4
    V4 -->|No| R3
    V4 -->|Yes| V5
    V5 -->|No| R3
    V5 -->|Yes| V6
    V6 -->|No| R3
    V6 -->|Yes| A1
    
    A1 --> A2
    A2 -->|No| R3
    A2 -->|Yes| A3
    A3 --> P1
    P1 --> P2
    P2 --> P3
    P3 --> R1
    
    R1 -->|Yes| R2
    R1 -->|No| R3
    R2 --> R4
    R3 --> R4
    
    style Input fill:#e3f2fd
    style Validation fill:#fff3e0
    style Auth fill:#e8f5e9
    style Processing fill:#f3e5f5
    style Response fill:#ffebee
```

### Security Measures

| Layer | Measure | Implementation |
|-------|---------|----------------|
| **Transport** | HTTPS | TLS/SSL encryption |
| **Authentication** | Bearer Token | Mercado Pago access token |
| **Validation** | Input sanitization | Pydantic validation |
| **Idempotency** | UUID keys | Prevent duplicate charges |
| **Data Protection** | Tokenization | Card data tokenized by MP |
| **Logging** | Sensitive data masking | No card numbers in logs |

---

## Payment Processing Flow

Detailed payment processing flow for each payment method:

```mermaid
flowchart TD
    subgraph Card["Credit Card Flow"]
        C1[User enters card data] --> C2[POST /create_payment]
        C2 --> C3[Validate card data]
        C3 --> C4[POST /v1/card_tokens]
        C4 --> C5[Receive card token]
        C5 --> C6[POST /v1/payments with token]
        C6 --> C7{Payment approved?}
        C7 -->|Yes| C8[Return success + card info]
        C7 -->|No| C9[Return rejection reason]
    end
    
    subgraph PIX["PIX Flow"]
        P1[User selects PIX] --> P2[POST /create_payment]
        P2 --> P3[Validate payer data]
        P3 --> P4[POST /v1/payments<br/>payment_method_id: pix]
        P4 --> P5[Generate QR Code]
        P5 --> P6[Return QR Code + ticket_url]
        P6 --> P7[User scans QR Code]
        P7 --> P8[Payment confirmation via webhook]
    end
    
    subgraph Boleto["Boleto Flow"]
        B1[User selects Boleto] --> B2[POST /create_payment]
        B2 --> B3[Validate payer + address data]
        B3 --> B4[POST /v1/payments<br/>payment_method_id: bolbradesco]
        B4 --> B5[Generate boleto]
        B5 --> B6[Return external_resource_url]
        B6 --> B7[User redirected to bank]
        B7 --> B8[Payment confirmation via webhook]
    end
    
    Card --> End
    PIX --> End
    Boleto --> End
    
    End([End])
    
    style Card fill:#e8f5e9
    style PIX fill:#e3f2fd
    style Boleto fill:#fff3e0
```

### Payment Status Transitions

```mermaid
stateDiagram-v2
    [*] --> pending: Payment created
    
    pending --> approved: Payment approved
    pending --> rejected: Payment rejected
    pending --> in_process: Under review
    pending --> cancelled: Cancelled
    
    in_process --> approved: Review completed
    in_process --> rejected: Review failed
    
    approved --> refunded: Refund issued
    approved --> charged_back: Chargeback
    
    rejected --> [*]: Final state
    cancelled --> [*]: Final state
    refunded --> [*]: Final state
    charged_back --> [*]: Final state
    approved --> [*]: Final state
    
    note right of pending
        Awaiting payment
        or bank processing
    end note
    
    note right of in_process
        Manual review
        or fraud check
    end note
    
    note right of approved
        Payment successful
        and accredited
    end note
```

---

## Component Diagram

UML Component diagram showing system modules and dependencies:

```mermaid
flowchart LR
    subgraph Frontend["Frontend Components"]
        HTML[checkout.html]
        JS[JavaScript Logic]
        CSS[Tailwind CSS]
    end
    
    subgraph Backend["Backend Components"]
        App[app.py - FastAPI]
        Routes[Route Handlers]
        Templates[Jinja2 Templates]
    end
    
    subgraph Services["Service Components"]
        MP[mercadopago.py]
        CardToken[Card Tokenization]
        Payment[Payment Processing]
        HTTP[HTTP Client]
    end
    
    subgraph Config["Configuration"]
        Env[Environment Variables]
        Decouple[python-decouple]
    end
    
    HTML --> JS
    JS --> CSS
    JS --> Routes
    
    Routes --> App
    App --> Templates
    App --> MP
    
    MP --> CardToken
    MP --> Payment
    MP --> HTTP
    MP --> Decouple
    
    Decouple --> Env
    
    CardToken --> HTTP
    Payment --> HTTP
    
    style Frontend fill:#e3f2fd
    style Backend fill:#e8f5e9
    style Services fill:#fff3e0
    style Config fill:#f3e5f5
```

---

## Deployment Diagram

Physical deployment architecture:

```mermaid
flowchart TB
    subgraph Internet["Internet"]
        User[End User]
        MP[Mercado Pago API]
    end
    
    subgraph CDN["CDN"]
        Tailwind[Tailwind CSS CDN]
    end
    
    subgraph Cloud["Cloud Platform"]
        LB[Load Balancer]
        
        subgraph AppServer["Application Server"]
            App1[FastAPI Instance 1]
            App2[FastAPI Instance 2]
        end
        
        subgraph Env["Environment"]
            Vars[Environment Variables]
            Secrets[Secrets Manager]
        end
    end
    
    User -->|HTTPS| LB
    LB --> App1
    LB --> App2
    
    App1 -->|HTTPS| MP
    App2 -->|HTTPS| MP
    
    App1 --> Tailwind
    App2 --> Tailwind
    
    App1 --> Vars
    App2 --> Vars
    
    Vars --> Secrets
    
    style Internet fill:#e3f2fd
    style CDN fill:#fff3e0
    style Cloud fill:#e8f5e9
    style AppServer fill:#f3e5f5
```

### Deployment Components

| Component | Purpose | Technology |
|-----------|---------|------------|
| **Load Balancer** | Traffic distribution | Nginx, AWS ALB |
| **Application Server** | FastAPI runtime | Uvicorn, Gunicorn |
| **Environment** | Configuration | Docker, Kubernetes |
| **Secrets Manager** | Credential storage | AWS Secrets, Vault |

---

## Next Steps

1. [Authentication & Security](authentication-security.md) - Security implementation details
2. [Development Guide](development.md) - Development workflow
3. [Deploy Guide](deploy.md) - Deployment instructions

---

**Last Updated**: April 2026  
**Version**: 1.0.0
