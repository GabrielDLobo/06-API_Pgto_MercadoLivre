# Guidelines & Standards

This document defines the coding standards, best practices, and guidelines for developing and maintaining the Mercado Libre Payment API.

---

## Table of Contents

- [Code Style](#code-style)
- [Python Guidelines](#python-guidelines)
- [API Design](#api-design)
- [Documentation Standards](#documentation-standards)
- [Git Workflow](#git-workflow)
- [Error Handling](#error-handling)
- [Security Guidelines](#security-guidelines)
- [Testing Standards](#testing-standards)
- [Frontend Guidelines](#frontend-guidelines)

---

## Code Style

### Python Style Guide

This project follows [PEP 8](https://pep8.org/) - Python Style Guide.

#### Key Conventions

| Aspect | Convention | Example |
|--------|------------|---------|
| **Indentation** | 4 spaces | `def function():` |
| **Line Length** | Max 79 chars | Use line breaks |
| **Naming** | snake_case | `def create_payment():` |
| **Classes** | PascalCase | `class MercadoPago:` |
| **Constants** | UPPER_CASE | `MAX_RETRIES = 5` |
| **Private** | Leading underscore | `_internal_method()` |

#### Example

```python
# ✅ Good
def create_payment(amount: float, description: str) -> dict:
    """Create a new payment transaction."""
    return {"amount": amount, "status": "approved"}


# ❌ Bad
def CreatePayment(amount,description):
    return {"amount":amount,"status":"approved"}
```

### Type Hints

Use type hints for all function parameters and return values:

```python
# ✅ Good - With type hints
def pay_with_card(
    amount: float,
    installments: int,
    description: str,
    card_data: dict,
    payer: dict,
) -> dict:
    pass


# ❌ Bad - No type hints
def pay_with_card(amount, installments, description, card_data, payer):
    pass
```

---

## Python Guidelines

### Import Order

Follow this import order:

1. Standard library imports
2. Third-party imports
3. Local application imports

```python
# ✅ Good
import uuid
from typing import Optional

import requests
from fastapi import FastAPI

from services import MercadoPago
```

### Docstrings

Use docstrings for all public functions, classes, and modules:

```python
# ✅ Good
def pay_with_pix(amount: float, description: str, payer: dict) -> dict:
    """
    Process a PIX payment transaction.

    Args:
        amount: Transaction amount in BRL
        description: Payment description
        payer: Payer information dictionary

    Returns:
        Payment response from Mercado Pago API

    Raises:
        RuntimeError: If API request fails
    """
    pass
```

### Error Messages

Use clear, descriptive error messages:

```python
# ✅ Good
raise HTTPException(status_code=400, detail="Invalid payment method provided")

# ❌ Bad
raise HTTPException(status_code=400, detail="Error")
```

---

## API Design

### RESTful Principles

| Principle | Implementation |
|-----------|----------------|
| **Resources** | Use nouns for endpoints |
| **HTTP Methods** | GET, POST, PUT, DELETE |
| **Status Codes** | Use appropriate HTTP status codes |
| **Versioning** | Include version in API paths |
| **Consistency** | Maintain consistent naming |

### Endpoint Naming

```
# ✅ Good
POST /create_payment
GET /payment/{id}

# ❌ Bad
POST /createPayment
GET /getPayment
```

### HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| `200` | OK | Successful GET/PUT requests |
| `201` | Created | Successful resource creation |
| `400` | Bad Request | Invalid input data |
| `401` | Unauthorized | Missing/invalid authentication |
| `404` | Not Found | Resource not found |
| `422` | Unprocessable Entity | Validation error |
| `500` | Internal Server Error | Server error |
| `502` | Bad Gateway | External service error |

### Response Format

```json
// ✅ Good - Consistent structure
{
    "status": "approved",
    "transaction_amount": 100.00,
    "payment_id": "123456789"
}

// ❌ Bad - Inconsistent
{
    "ok": true,
    "data": {...}
}
```

---

## Documentation Standards

### Code Comments

Use comments sparingly and focus on **why**, not **what**:

```python
# ✅ Good - Explains why
# Using UUID for idempotency to prevent duplicate charges
headers['X-Idempotency-Key'] = str(uuid.uuid4())

# ❌ Bad - States obvious
# Set the header
headers['X-Idempotency-Key'] = str(uuid.uuid4())
```

### README Structure

```markdown
# Project Name

Brief description

## Features

- Feature 1
- Feature 2

## Installation

Steps to install

## Usage

How to use

## API Reference

Endpoints documentation
```

### API Documentation

Document all endpoints with:

- Endpoint path
- HTTP method
- Request parameters
- Request body schema
- Response schema
- Error responses
- Example requests

---

## Git Workflow

### Branch Naming

```
# Feature branches
feature/add-boleto-payment
feature/improve-error-handling

# Bug fixes
bugfix/fix-pix-validation
bugfix/resolve-token-expiry

# Documentation
docs/add-api-examples
docs/update-installation-guide
```

### Commit Messages

Follow conventional commits:

```
# Format
<type>(<scope>): <subject>

# Types
feat: New feature
fix: Bug fix
docs: Documentation
style: Formatting
refactor: Code refactoring
test: Tests
chore: Maintenance
```

#### Examples

```bash
# ✅ Good
feat(payment): add boleto payment support
fix(api): resolve 404 error on payment status
docs(readme): update installation instructions

# ❌ Bad
update code
fixed bug
stuff
```

### Pull Request Guidelines

1. **Title**: Clear and descriptive
2. **Description**: Explain changes and motivation
3. **Testing**: Include test results
4. **Screenshots**: For UI changes
5. **Checklist**: Verify all requirements

---

## Error Handling

### Exception Hierarchy

```python
# ✅ Good - Specific exceptions
try:
    response.raise_for_status()
except requests.HTTPError as http_err:
    raise HTTPException(status_code=502, detail=f"API Error: {http_err}")
except requests.ConnectionError as conn_err:
    raise HTTPException(status_code=503, detail="Service unavailable")
except Exception as err:
    raise HTTPException(status_code=500, detail="Internal server error")
```

### Error Response Format

```json
{
    "detail": "Invalid payment method",
    "status_code": 400,
    "timestamp": "2026-04-01T10:00:00Z"
}
```

### Validation Errors

```python
# ✅ Good - Descriptive validation
if amount <= 0:
    raise HTTPException(
        status_code=400,
        detail="Transaction amount must be greater than zero"
    )
```

---

## Security Guidelines

### Input Validation

```python
# ✅ Good - Validate all inputs
async def create_payment(request: Request):
    data = await request.json()
    
    # Validate required fields
    if 'payment_method' not in data:
        raise HTTPException(status_code=400, detail="Payment method required")
    
    # Validate amount
    amount = float(data.get('transaction_amount', 0))
    if amount <= 0:
        raise HTTPException(status_code=400, detail="Invalid amount")
```

### Sensitive Data

```python
# ✅ Good - Never log sensitive data
logger.info(f"Payment processed for amount: {amount}")

# ❌ Bad - Logging sensitive information
logger.info(f"Payment with card: {card_number}")  # NEVER DO THIS
```

### Authentication

```python
# ✅ Good - Use environment variables
from decouple import config

ACCESS_TOKEN = config('MP_ACCESS_TOKEN')

# ❌ Bad - Hardcoded credentials
ACCESS_TOKEN = "APP_USR-xxx"  # NEVER DO THIS
```

---

## Testing Standards

### Test Naming

```python
# ✅ Good - Descriptive names
def test_create_payment_with_valid_card_data():
    pass

def test_reject_payment_with_expired_card():
    pass

# ❌ Bad - Vague names
def test_payment():
    pass
```

### Test Structure (AAA Pattern)

```python
# ✅ Good - Arrange, Act, Assert
def test_pix_payment_creation():
    # Arrange
    mp = MercadoPago()
    amount = 100.00
    
    # Act
    result = mp.pay_with_pix(amount, "Test", payer)
    
    # Assert
    assert result['status'] == 'pending'
    assert result['payment_method'] == 'pix'
```

### Test Coverage

Aim for minimum coverage:

| Component | Minimum Coverage |
|-----------|------------------|
| **Services** | 90% |
| **API Endpoints** | 85% |
| **Utilities** | 80% |
| **Overall** | 85% |

---

## Frontend Guidelines

### HTML Structure

```html
<!-- ✅ Good - Semantic HTML -->
<form id="payment-form" class="space-y-4">
    <label for="card-number">Card Number</label>
    <input type="text" id="card-number" name="card_number" required>
</form>

<!-- ❌ Bad - Non-semantic -->
<div>
    <span>Card Number</span>
    <input name="card_number">
</div>
```

### CSS Classes (Tailwind)

```html
<!-- ✅ Good - Consistent Tailwind usage -->
<button class="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700">
    Pay
</button>

<!-- ❌ Bad - Inconsistent spacing -->
<button class="bg-blue-600 text-white p-2 rounded hover:bg-blue-700 w-full">
    Pay
</button>
```

### JavaScript

```javascript
// ✅ Good - Async/await
async function createPayment(data) {
    try {
        const response = await fetch('/create_payment', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        });
        return await response.json();
    } catch (error) {
        console.error('Payment failed:', error);
        throw error;
    }
}

// ❌ Bad - Callback hell
function createPayment(data, callback) {
    fetch('/create_payment', {...}).then(r => r.json()).then(d => callback(d));
}
```

---

## Code Review Checklist

Before submitting code:

- [ ] Follows PEP 8 style guide
- [ ] Includes type hints
- [ ] Has docstrings for public functions
- [ ] Error handling implemented
- [ ] Tests written and passing
- [ ] No sensitive data in code
- [ ] Git commit message follows convention
- [ ] Documentation updated

---

## Tools & Linters

### Recommended Tools

| Tool | Purpose | Configuration |
|------|---------|---------------|
| **black** | Code formatting | Default settings |
| **flake8** | Linting | Max line length: 79 |
| **mypy** | Type checking | Strict mode |
| **isort** | Import sorting | Default profile |
| **pre-commit** | Git hooks | Run on commit |

### Pre-commit Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.0.0
    hooks:
      - id: black
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
  - repo: https://github.com/pycqa/isort
    rev: 5.13.0
    hooks:
      - id: isort
```

---

## Next Steps

1. [Project Structure](project-structure.md) - Understand file organization
2. [Development Guide](development.md) - Start developing
3. [Testing Guide](testing.md) - Write and run tests

---

**Last Updated**: April 2026  
**Version**: 1.0.0
