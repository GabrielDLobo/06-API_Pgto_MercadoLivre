# Authentication & Security

This document details the authentication mechanisms, security measures, and best practices implemented in the Mercado Libre Payment API.

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Authorization](#authorization)
- [Data Protection](#data-protection)
- [Secure Communication](#secure-communication)
- [Input Validation](#input-validation)
- [Error Handling](#error-handling)
- [Logging & Auditing](#logging--auditing)
- [Security Best Practices](#security-best-practices)
- [Compliance](#compliance)

---

## Overview

The API implements multiple security layers to protect sensitive payment data and ensure secure transactions:

| Security Layer | Implementation |
|----------------|----------------|
| **Authentication** | Bearer Token (Mercado Pago) |
| **Transport Security** | HTTPS/TLS |
| **Data Protection** | Tokenization, Encryption |
| **Request Security** | Idempotency Keys |
| **Input Validation** | Type checking, Sanitization |

---

## Authentication

### Mercado Pago Authentication

The API authenticates with Mercado Pago using OAuth 2.0 Bearer tokens.

#### Token Types

| Token Type | Prefix | Usage |
|------------|--------|-------|
| **Production** | `APP_USR-` | Live transactions |
| **Sandbox** | `TEST-` | Testing only |

#### Authentication Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant MP as Mercado Pago
    participant App as Application
    participant Env as Environment
    
    Dev->>MP: Create application in dashboard
    MP-->>Dev: Generate Access Token
    Dev->>Env: Store token in .env file
    Env->>App: Load token at startup
    
    loop Every Request
        App->>App: Read token from environment
        App->>App: Add to Authorization header
        App->>MP: POST request with Bearer token
        MP->>MP: Validate token
        MP-->>App: Response (200/401)
    end
```

#### Implementation

```python
# services/mercadopago.py
from decouple import config

class MercadoPago:
    def __init__(self):
        self.__access_token = config('MP_ACCESS_TOKEN')
        self.__headers = {
            'Content-Type': 'application/json',
            'Authorization': f'Bearer {self.__access_token}',
        }
```

#### Security Headers

```http
POST /v1/payments HTTP/1.1
Host: api.mercadopago.com
Authorization: Bearer APP_USR-xxxxxxxxxxxxxxxx
Content-Type: application/json
X-Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

---

## Authorization

### Access Control

The API implements the following access control measures:

| Control | Description |
|---------|-------------|
| **Token-based** | All requests require valid access token |
| **Scope-based** | Token permissions limit available actions |
| **Environment isolation** | Sandbox and production tokens are separate |

### Token Permissions

Mercado Pago tokens can have different scopes:

| Scope | Permission |
|-------|------------|
| `payment` | Create and manage payments |
| `refund` | Process refunds |
| `customer` | Manage customer data |
| `merchant_orders` | Access merchant orders |

---

## Data Protection

### Sensitive Data Handling

#### Data Classification

| Data Type | Sensitivity | Protection Method |
|-----------|-------------|-------------------|
| Card Number | Critical | Tokenization |
| CVV/CVC | Critical | Never stored |
| CPF | High | Encrypted transmission |
| Email | Medium | Access control |
| Transaction Amount | Medium | Access control |

#### Card Tokenization

Credit card data is tokenized before processing:

```mermaid
flowchart LR
    A[Raw Card Data] --> B[POST /v1/card_tokens]
    B --> C[Mercado Pago API]
    C --> D[Card Token]
    D --> E[Use token in payment]
    
    style A fill:#ffebee
    style D fill:#e8f5e9
```

**Implementation:**

```python
def __get_card_token(self, card_data: dict) -> str:
    """Tokenize card data - raw card number never stored."""
    response = self.__post('/v1/card_tokens', card_data)
    return response.get('id')  # Returns token, not card data

def pay_with_card(self, amount: float, installments: int, 
                  description: str, card_data: dict, payer: dict) -> dict:
    """Create payment using tokenized card."""
    token = self.__get_card_token(card_data)  # Get token first
    payload = {
        'transaction_amount': amount,
        'token': token,  # Use token, not raw card data
        'description': description,
        'installments': int(installments),
        'payer': payer,
    }
    return self.__create_payment(payload)
```

#### Data Storage Rules

| Data | Store? | Reason |
|------|--------|--------|
| Card Number | ❌ No | PCI-DSS compliance |
| CVV | ❌ No | Never store security codes |
| Card Token | ✅ Yes | Safe to store |
| Last 4 digits | ✅ Yes | For display purposes |
| Card Brand | ✅ Yes | Non-sensitive |

---

## Secure Communication

### HTTPS/TLS

All communication must use HTTPS with TLS 1.2 or higher:

```mermaid
flowchart LR
    Client[Client] -->|HTTPS TLS 1.3| Server[API Server]
    Server -->|HTTPS TLS 1.3| MP[Mercado Pago]
    
    style Client fill:#e3f2fd
    style Server fill:#e8f5e9
    style MP fill:#fff3e0
```

### TLS Configuration

| Setting | Value |
|---------|-------|
| **Minimum Version** | TLS 1.2 |
| **Recommended** | TLS 1.3 |
| **Cipher Suites** | Strong ciphers only |
| **Certificate** | Valid SSL certificate required |

### Idempotency

Prevent duplicate transactions with idempotency keys:

```python
def __post(self, path: str, payload: dict) -> dict:
    url = f'{self.BASE_URL}{path}'
    # Generate unique idempotency key for each request
    headers = {**self.__headers, 'X-Idempotency-Key': str(uuid.uuid4())}
    response = requests.post(url=url, json=payload, headers=headers)
    return response.json()
```

**Benefits:**
- Prevents duplicate charges on network retries
- Ensures payment safety on connection failures
- Provides transaction traceability

---

## Input Validation

### Request Validation

All incoming requests are validated:

```python
@app.post('/create_payment')
async def create_payment(request: Request):
    data = await request.json()
    
    # Validate payment method
    method = data.get('payment_method')
    if method not in ['card', 'pix', 'boleto']:
        raise HTTPException(status_code=400, 
                          detail='Invalid payment method')
    
    # Validate amount
    amount = float(data.get('transaction_amount', 0))
    if amount <= 0:
        raise HTTPException(status_code=400, 
                          detail='Amount must be greater than zero')
    
    # Validate required fields based on payment method
    if method == 'card':
        self._validate_card_data(data)
    elif method == 'boleto':
        self._validate_boleto_data(data)
```

### Validation Rules

| Field | Validation | Error Message |
|-------|------------|---------------|
| `payment_method` | Enum: card, pix, boleto | "Invalid payment method" |
| `transaction_amount` | Number > 0 | "Amount must be greater than zero" |
| `email` | Valid email format | "Invalid email format" |
| `card_number` | Luhn algorithm | "Invalid card number" |
| `security_code` | 3-4 digits | "Invalid security code" |
| `identification_number` | CPF validation | "Invalid CPF" |
| `expiration_month` | 01-12 | "Invalid expiration month" |
| `expiration_year` | Current year or later | "Card expired" |

### Sanitization

```python
# String sanitization
def sanitize_string(value: str) -> str:
    """Remove potentially harmful characters."""
    if not isinstance(value, str):
        return value
    # Remove null bytes and control characters
    return ''.join(char for char in value if ord(char) >= 32)
```

---

## Error Handling

### Secure Error Responses

Error messages don't expose sensitive information:

```python
# ✅ Good - Generic error message
try:
    result = mp.pay_with_card(...)
except RuntimeError as err:
    logger.error(f"Payment processing error: {err}")
    raise HTTPException(status_code=502, detail="Payment processing failed")

# ❌ Bad - Exposes internal details  
except RuntimeError as err:
    raise HTTPException(status_code=502, detail=f"Error: {err}")
```

### Error Code Mapping

| Internal Error | HTTP Status | User Message |
|----------------|-------------|--------------|
| Validation Error | 400 | "Invalid payment data" |
| Authentication Error | 401 | "Authentication failed" |
| Resource Not Found | 404 | "Payment not found" |
| Mercado Pago Error | 502 | "Payment service unavailable" |
| Internal Error | 500 | "Internal server error" |

---

## Logging & Auditing

### Secure Logging

Log transactions without sensitive data:

```python
import logging

logger = logging.getLogger(__name__)

# ✅ Good - Safe logging
logger.info(f"Payment created: method={method}, amount={amount}, status={status}")

# ❌ Bad - Never log sensitive data
logger.info(f"Payment with card={card_number}, cvv={security_code}")  # NEVER!
```

### Audit Trail

| Event | Logged Information |
|-------|-------------------|
| Payment Created | Timestamp, method, amount, status |
| Payment Approved | Timestamp, payment ID, status |
| Payment Rejected | Timestamp, reason (generic) |
| API Error | Timestamp, endpoint, error code |

### Log Retention

| Log Type | Retention Period |
|----------|------------------|
| Transaction logs | 5 years (regulatory) |
| Access logs | 1 year |
| Error logs | 1 year |
| Audit logs | 5 years |

---

## Security Best Practices

### DO's ✅

1. **Use environment variables** for all credentials
2. **Enable HTTPS** in production
3. **Validate all inputs** before processing
4. **Use idempotency keys** for payment requests
5. **Log transactions** without sensitive data
6. **Rotate access tokens** periodically
7. **Use sandbox tokens** for development
8. **Implement rate limiting**
9. **Monitor for suspicious activity**
10. **Keep dependencies updated**

### DON'Ts ❌

1. **Never commit** `.env` files to version control
2. **Never log** card numbers or CVV
3. **Never store** raw card data
4. **Never use** production tokens in development
5. **Never disable** SSL/TLS verification
6. **Never expose** tokens in client-side code
7. **Never share** access tokens
8. **Never skip** input validation
9. **Never ignore** security warnings
10. **Never hardcode** credentials

### Security Checklist

Before deploying to production:

- [ ] HTTPS enabled with valid certificate
- [ ] Production access token configured
- [ ] Debug mode disabled (`DEBUG=False`)
- [ ] Environment variables secured
- [ ] Input validation implemented
- [ ] Error handling doesn't expose internals
- [ ] Logging excludes sensitive data
- [ ] Rate limiting configured
- [ ] Firewall rules in place
- [ ] Monitoring and alerting configured

---

## Compliance

### PCI-DSS Compliance

The API follows PCI-DSS guidelines by:

| Requirement | Implementation |
|-------------|----------------|
| **No card storage** | Card data tokenized by Mercado Pago |
| **Secure transmission** | HTTPS/TLS for all communication |
| **Access control** | Token-based authentication |
| **Logging** | Audit trail without sensitive data |
| **Network security** | Firewall and security groups |

### LGPD (Brazilian GDPR)

Data protection compliance:

| Principle | Implementation |
|-----------|----------------|
| **Data minimization** | Only collect necessary data |
| **Purpose limitation** | Data used only for payment processing |
| **Security** | Encryption and access controls |
| **Retention** | Data kept only as long as required |

### Security Standards Reference

| Standard | Description |
|----------|-------------|
| **PCI-DSS** | Payment Card Industry Data Security Standard |
| **LGPD** | Lei Geral de Proteção de Dados (Brazil) |
| **OWASP** | Open Web Application Security Project |

---

## Threat Mitigation

### Common Threats

| Threat | Mitigation |
|--------|------------|
| **Man-in-the-Middle** | HTTPS/TLS encryption |
| **Replay Attacks** | Idempotency keys |
| **SQL Injection** | No direct SQL (using API) |
| **XSS** | Jinja2 auto-escaping |
| **CSRF** | Same-site cookies, CORS |
| **Brute Force** | Rate limiting |
| **Data Leakage** | Tokenization, logging controls |

### Security Architecture

```mermaid
flowchart TB
    subgraph Perimeter["Security Perimeter"]
        Firewall[Firewall/WAF]
        LB[Load Balancer with SSL]
    end
    
    subgraph Application["Application Security"]
        Auth[Authentication Layer]
        Validation[Input Validation]
        RateLimit[Rate Limiting]
    end
    
    subgraph Data["Data Security"]
        Tokenization[Card Tokenization]
        Encryption[Data Encryption]
        Masking[Response Masking]
    end
    
    User --> Firewall
    Firewall --> LB
    LB --> Auth
    Auth --> Validation
    Validation --> RateLimit
    RateLimit --> Tokenization
    Tokenization --> Encryption
    Encryption --> Masking
    
    style Perimeter fill:#ffebee
    style Application fill:#fff3e0
    style Data fill:#e8f5e9
```

---

## Next Steps

1. [Development Guide](development.md) - Secure development practices
2. [Testing Guide](testing.md) - Security testing procedures
3. [Deploy Guide](deploy.md) - Secure deployment configuration

---

**Last Updated**: April 2026  
**Version**: 1.0.0
