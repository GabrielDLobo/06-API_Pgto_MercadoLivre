# API Endpoints

This document provides a comprehensive reference for all API endpoints available in the Mercado Libre Payment API.

---

## Table of Contents

- [Overview](#overview)
- [Base URL](#base-url)
- [Endpoints](#endpoints)
  - [Checkout Page](#checkout-page)
  - [Create Payment](#create-payment)
- [Payment Methods](#payment-methods)
- [Request/Response Examples](#requestresponse-examples)
- [Error Codes](#error-codes)
- [Rate Limiting](#rate-limiting)

---

## Overview

The API provides endpoints for processing payments through Mercado Pago. All endpoints follow RESTful conventions and return JSON responses.

| Aspect | Details |
|--------|---------|
| **Protocol** | HTTP/HTTPS |
| **Content Type** | `application/json` |
| **Character Encoding** | UTF-8 |
| **Authentication** | Bearer Token (via service layer) |

---

## Base URL

| Environment | URL |
|-------------|-----|
| **Development** | `http://localhost:8000` |
| **Production** | `https://your-domain.com` |

---

## Endpoints

### Checkout Page

Renders the payment checkout HTML page.

```
GET /
```

**Headers:**
| Header | Value | Required |
|--------|-------|----------|
| `Accept` | `text/html` | Yes |

**Response:**
- **Content-Type**: `text/html; charset=utf-8`
- **Status**: `200 OK`

**Example:**
```bash
curl -X GET http://localhost:8000/
```

**Response:** HTML page with payment form

---

### Create Payment

Creates a new payment transaction with Mercado Pago.

```
POST /create_payment
```

**Headers:**
| Header | Value | Required |
|--------|-------|----------|
| `Content-Type` | `application/json` | Yes |

**Request Body:**

The request body varies based on the payment method selected.

#### Common Fields (All Payment Methods)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `payment_method` | string | ✅ | Payment method: `card`, `pix`, or `boleto` |
| `transaction_amount` | number | ✅ | Transaction amount in BRL |
| `description` | string | ✅ | Payment description |
| `email` | string | ✅ | Payer's email address |

---

### Payment Method: Credit Card

**Additional Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `card_number` | string | Credit card number |
| `expiration_month` | string | Expiration month (MM) |
| `expiration_year` | string | Expiration year (YYYY) |
| `security_code` | string | CVV/CVC code |
| `cardholder_name` | string | Cardholder's full name |
| `identification_number` | string | CPF (Brazilian tax ID) |
| `installments` | integer | Number of installments |

**Example Request:**
```json
{
  "payment_method": "card",
  "transaction_amount": 100.00,
  "description": "Test Payment",
  "email": "test@example.com",
  "card_number": "5031755734530604",
  "expiration_month": "11",
  "expiration_year": "2025",
  "security_code": "123",
  "cardholder_name": "John Doe",
  "identification_number": "12345678900",
  "installments": 1
}
```

**Example Response (Success):**
```json
{
  "id": "12345678901",
  "status": "approved",
  "status_detail": "accredited",
  "transaction_amount": 100.00,
  "payment_method_id": "visa",
  "date_created": "2026-04-01T10:00:00.000-03:00",
  "date_approved": "2026-04-01T10:00:01.000-03:00",
  "payer": {
    "email": "test@example.com",
    "identification": {
      "type": "CPF",
      "number": "12345678900"
    }
  },
  "card": {
    "first_six_digits": "503175",
    "last_four_digits": "0604",
    "cardholder": {
      "name": "John Doe"
    }
  }
}
```

---

### Payment Method: PIX

**Additional Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `identification_number` | string | CPF (Brazilian tax ID) |

**Example Request:**
```json
{
  "payment_method": "pix",
  "transaction_amount": 100.00,
  "description": "Test PIX Payment",
  "email": "test@example.com",
  "identification_number": "12345678900"
}
```

**Example Response (Success):**
```json
{
  "id": "12345678902",
  "status": "pending",
  "status_detail": "pending_payment",
  "transaction_amount": 100.00,
  "payment_method_id": "pix",
  "date_created": "2026-04-01T10:00:00.000-03:00",
  "point_of_interaction": {
    "transaction_data": {
      "qr_code": "00020126580014BR.GOV.BCB.PIX...",
      "qr_code_base64": "iVBORw0KGgoAAAANSUhEUgAA...",
      "ticket_url": "https://www.mercadopago.com.br/qr/..."
    }
  },
  "payer": {
    "email": "test@example.com",
    "identification": {
      "type": "CPF",
      "number": "12345678900"
    }
  }
}
```

---

### Payment Method: Boleto

**Additional Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `first_name` | string | Payer's first name |
| `last_name` | string | Payer's last name |
| `identification_number` | string | CPF (Brazilian tax ID) |
| `zip_code` | string | ZIP/Postal code |
| `street_name` | string | Street name |
| `street_number` | string | Street number |
| `neighborhood` | string | Neighborhood |
| `city` | string | City |
| `federal_unit` | string | State (UF) - 2 letters |

**Example Request:**
```json
{
  "payment_method": "boleto",
  "transaction_amount": 100.00,
  "description": "Test Boleto Payment",
  "email": "test@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "identification_number": "12345678900",
  "zip_code": "01001-000",
  "street_name": "Praça da Sé",
  "street_number": "100",
  "neighborhood": "Sé",
  "city": "São Paulo",
  "federal_unit": "SP"
}
```

**Example Response (Success):**
```json
{
  "id": "12345678903",
  "status": "pending",
  "status_detail": "pending_payment",
  "transaction_amount": 100.00,
  "payment_method_id": "bolbradesco",
  "date_created": "2026-04-01T10:00:00.000-03:00",
  "transaction_details": {
    "external_resource_url": "https://www.mercadopago.com.br/boleto/..."
  },
  "payer": {
    "email": "test@example.com",
    "identification": {
      "type": "CPF",
      "number": "12345678900"
    }
  }
}
```

---

## Payment Methods

### Summary

| Method | Code | Processing Time | Features |
|--------|------|-----------------|----------|
| **Credit Card** | `card` | Instant | Installments, Tokenization |
| **PIX** | `pix` | Instant | QR Code, Copy & Paste |
| **Boleto** | `boleto` | 1-3 business days | Bank Slip, External URL |

### Supported Card Brands

- Visa
- Mastercard
- Elo
- American Express
- Hipercard
- Aura

### Installment Options

| Card Type | Minimum | Maximum |
|-----------|---------|---------|
| **Visa/Mastercard** | 1 | 12 |
| **Elo** | 1 | 12 |
| **Amex** | 1 | 6 |

---

## Request/Response Examples

### cURL Examples

#### Credit Card Payment

```bash
curl -X POST http://localhost:8000/create_payment \
  -H "Content-Type: application/json" \
  -d '{
    "payment_method": "card",
    "transaction_amount": 100.00,
    "description": "Test Payment",
    "email": "test@example.com",
    "card_number": "5031755734530604",
    "expiration_month": "11",
    "expiration_year": "2025",
    "security_code": "123",
    "cardholder_name": "John Doe",
    "identification_number": "12345678900",
    "installments": 1
  }'
```

#### PIX Payment

```bash
curl -X POST http://localhost:8000/create_payment \
  -H "Content-Type: application/json" \
  -d '{
    "payment_method": "pix",
    "transaction_amount": 50.00,
    "description": "PIX Test",
    "email": "pix@example.com",
    "identification_number": "12345678900"
  }'
```

#### Boleto Payment

```bash
curl -X POST http://localhost:8000/create_payment \
  -H "Content-Type: application/json" \
  -d '{
    "payment_method": "boleto",
    "transaction_amount": 75.50,
    "description": "Boleto Test",
    "email": "boleto@example.com",
    "first_name": "Jane",
    "last_name": "Smith",
    "identification_number": "98765432100",
    "zip_code": "20000-000",
    "street_name": "Rua do Comércio",
    "street_number": "500",
    "neighborhood": "Centro",
    "city": "Rio de Janeiro",
    "federal_unit": "RJ"
  }'
```

### JavaScript (Fetch API)

```javascript
async function createPayment(paymentData) {
  const response = await fetch('/create_payment', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(paymentData),
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.detail);
  }
  
  return await response.json();
}

// Usage
const payment = {
  payment_method: 'card',
  transaction_amount: 100.00,
  description: 'Test Payment',
  email: 'test@example.com',
  card_number: '5031755734530604',
  expiration_month: '11',
  expiration_year: '2025',
  security_code: '123',
  cardholder_name: 'John Doe',
  identification_number: '12345678900',
  installments: 1,
};

createPayment(payment)
  .then(result => console.log('Payment created:', result))
  .catch(error => console.error('Payment failed:', error));
```

### Python (Requests)

```python
import requests

def create_payment(payment_data):
    response = requests.post(
        'http://localhost:8000/create_payment',
        json=payment_data,
        headers={'Content-Type': 'application/json'}
    )
    response.raise_for_status()
    return response.json()

# Usage
payment = {
    'payment_method': 'pix',
    'transaction_amount': 100.00,
    'description': 'Test Payment',
    'email': 'test@example.com',
    'identification_number': '12345678900'
}

result = create_payment(payment)
print(f"Payment ID: {result['id']}")
```

---

## Error Codes

### HTTP Status Codes

| Status Code | Meaning | Description |
|-------------|---------|-------------|
| `200` | OK | Request successful |
| `400` | Bad Request | Invalid payment data |
| `401` | Unauthorized | Invalid credentials |
| `404` | Not Found | Resource not found |
| `422` | Unprocessable Entity | Validation error |
| `500` | Internal Server Error | Server error |
| `502` | Bad Gateway | Mercado Pago API error |

### Error Response Format

```json
{
  "detail": "Error description here"
}
```

### Common Errors

| Error | Status | Cause | Solution |
|-------|--------|-------|----------|
| `Invalid payment method` | 400 | Unknown payment_method value | Use: card, pix, or boleto |
| `Invalid amount` | 400 | Amount <= 0 or invalid format | Use positive decimal number |
| `Missing required field` | 400 | Required field not provided | Check required fields for method |
| `Invalid card number` | 400 | Card number failed validation | Use valid test card numbers |
| `Erro MP 401` | 502 | Invalid access token | Check MP_ACCESS_TOKEN |
| `Erro MP 400` | 502 | Mercado Pago validation error | Review payment data |

---

## Rate Limiting

### Mercado Pago API Limits

| Limit Type | Value |
|------------|-------|
| **Requests per minute** | 100 |
| **Requests per hour** | 1000 |
| **Requests per day** | 10000 |

### Best Practices

1. **Implement retry logic** with exponential backoff
2. **Cache responses** when appropriate
3. **Use idempotency keys** for payment creation
4. **Monitor API usage** to avoid hitting limits

### Retry Strategy

```python
import time
from requests.exceptions import RequestException

def create_payment_with_retry(payment_data, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = mp.pay_with_pix(
                amount=payment_data['amount'],
                description=payment_data['description'],
                payer=payment_data['payer']
            )
            return result
        except RequestException as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt  # Exponential backoff
            time.sleep(wait_time)
```

---

## API Documentation (Interactive)

The API includes interactive documentation:

| Documentation | URL |
|---------------|-----|
| **Swagger UI** | `http://localhost:8000/docs` |
| **ReDoc** | `http://localhost:8000/redoc` |

### Swagger UI Features

- Interactive API testing
- Request/response schema visualization
- Try-it-out functionality
- Authentication management

---

## Next Steps

1. [System Modeling](system-modeling.md) - Architecture diagrams
2. [Authentication & Security](authentication-security.md) - Security implementation
3. [Testing Guide](testing.md) - Test payment flows

---

**Last Updated**: April 2026  
**Version**: 1.0.0
