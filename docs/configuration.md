# Configuration Guide

This guide covers all configuration aspects of the Mercado Libre Payment API, including environment variables, Mercado Pago credentials, and application settings.

---

## Table of Contents

- [Environment Variables](#environment-variables)
- [Mercado Pago Credentials](#mercado-pago-credentials)
- [Application Settings](#application-settings)
- [Development vs Production](#development-vs-production)
- [Configuration Best Practices](#configuration-best-practices)

---

## Environment Variables

The application uses environment variables for configuration, managed through the `python-decouple` library.

### Required Variables

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `MP_BASE_API_URL` | Mercado Pago API base URL | `https://api.mercadopago.com` | ✅ |
| `MP_ACCESS_TOKEN` | Mercado Pago access token | `APP_USR-xxx-xxx` | ✅ |

### Optional Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `APP_HOST` | Server host | `0.0.0.0` | `127.0.0.1` |
| `APP_PORT` | Server port | `8000` | `8080` |
| `DEBUG` | Debug mode | `False` | `True` |

---

## Mercado Pago Credentials

### Obtaining Credentials

#### Step 1: Create Mercado Pago Account

1. Visit [Mercado Pago](https://www.mercadopago.com.br/)
2. Click "Create Account"
3. Complete registration with your information

#### Step 2: Access Developer Dashboard

1. Go to [Mercado Pago Developers](https://developers.mercadopago.com/)
2. Log in with your Mercado Pago credentials
3. Navigate to "Your Applications"

#### Step 3: Create New Application

1. Click "Create Application"
2. Select application type: "Payment processing"
3. Fill in application details:
   - **Name**: Your application name
   - **Description**: Brief description
   - **Logo**: Upload application logo (optional)

#### Step 4: Get Credentials

After creating the application:

1. Navigate to "Credentials" tab
2. You'll find:
   - **Access Token** (Production)
   - **Public Key** (Production)
   - **Access Token** (Sandbox)
   - **Public Key** (Sandbox)

### Environment Configuration

Create a `.env` file in the project root:

```env
# =============================================================================
# MERCADO PAGO API CONFIGURATION
# =============================================================================

# Production API URL
MP_BASE_API_URL=https://api.mercadopago.com

# Production Access Token (for live transactions)
MP_ACCESS_TOKEN=APP_USR-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# =============================================================================
# APPLICATION SETTINGS
# =============================================================================

# Server configuration
APP_HOST=0.0.0.0
APP_PORT=8000

# Debug mode (set to False in production)
DEBUG=True
```

### Sandbox Mode (Testing)

For testing without real transactions, use Sandbox credentials:

```env
# Sandbox Mode
MP_BASE_API_URL=https://api.mercadopago.com
MP_ACCESS_TOKEN=TEST-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

> **Note**: Sandbox tokens start with `TEST-` prefix

---

## Application Settings

### Server Configuration

| Setting | Description | Development | Production |
|---------|-------------|-------------|------------|
| `APP_HOST` | Network interface | `0.0.0.0` | `0.0.0.0` |
| `APP_PORT` | Port number | `8000` | `80` or `443` |
| `DEBUG` | Debug mode | `True` | `False` |

### Recommended Configuration

#### Development Environment

```env
APP_HOST=127.0.0.1
APP_PORT=8000
DEBUG=True
```

#### Production Environment

```env
APP_HOST=0.0.0.0
APP_PORT=8000
DEBUG=False
```

---

## Development vs Production

### Configuration Comparison

| Aspect | Development | Production |
|--------|-------------|------------|
| **Access Token** | Sandbox token | Production token |
| **Debug Mode** | Enabled | Disabled |
| **Host** | localhost | 0.0.0.0 |
| **Port** | 8000 | 80/443 (via proxy) |
| **HTTPS** | Optional | Required |
| **Logging** | Verbose | Minimal |

### Environment-Specific Files

Recommended structure:

```
.env                 # Local overrides (gitignored)
.env.example         # Template for .env (committed)
.env.production      # Production settings (secure storage)
.env.development     # Development settings
```

### .env.example Template

```env
# Copy this file to .env and fill in your values

# Mercado Pago Configuration
MP_BASE_API_URL=https://api.mercadopago.com
MP_ACCESS_TOKEN=your-access-token-here

# Application Settings
APP_HOST=0.0.0.0
APP_PORT=8000
DEBUG=False
```

---

## Configuration Best Practices

### Security Guidelines

#### ✅ DO:

1. **Use environment variables** for all sensitive data
2. **Keep `.env` files out of version control** (already in `.gitignore`)
3. **Use different tokens** for development and production
4. **Rotate access tokens** periodically
5. **Use HTTPS** in production environments
6. **Validate all configuration** on application startup

#### ❌ DON'T:

1. **Never commit** `.env` files to Git
2. **Never share** access tokens publicly
3. **Never use** production tokens in development
4. **Never hardcode** credentials in source code
5. **Never log** sensitive configuration values

### Configuration Validation

Add validation on application startup:

```python
# Example validation logic
from decouple import config

MP_ACCESS_TOKEN = config('MP_ACCESS_TOKEN')

if not MP_ACCESS_TOKEN or MP_ACCESS_TOKEN == 'your-access-token-here':
    raise ValueError("MP_ACCESS_TOKEN must be configured")
```

### Secret Management

For production deployments, consider:

| Platform | Secret Management |
|----------|-------------------|
| **AWS** | AWS Secrets Manager / Parameter Store |
| **Azure** | Azure Key Vault |
| **GCP** | Google Secret Manager |
| **Heroku** | Config Vars |
| **Railway** | Environment Variables |
| **Render** | Environment Variables |

---

## Configuration Checklist

Before deploying to production:

- [ ] Replace sandbox token with production token
- [ ] Set `DEBUG=False`
- [ ] Configure proper `APP_HOST` and `APP_PORT`
- [ ] Enable HTTPS
- [ ] Set up proper logging
- [ ] Configure firewall rules
- [ ] Test all payment methods
- [ ] Verify webhook configuration
- [ ] Review security settings

---

## Troubleshooting Configuration

### Common Issues

#### 1. Missing Environment Variable

**Error**: `Required configuration variable not set`

**Solution**: Ensure all required variables are in `.env`:

```bash
# Check if .env exists
ls -la .env

# Verify variable values
cat .env
```

#### 2. Invalid Access Token

**Error**: `401 Unauthorized` from Mercado Pago API

**Solution**:
1. Verify token is correct
2. Check if token has expired
3. Ensure using correct environment (sandbox vs production)

#### 3. Configuration Not Loading

**Error**: Settings not being applied

**Solution**:
1. Restart the application after `.env` changes
2. Verify `python-decouple` is installed
3. Check `.env` file encoding (UTF-8)

---

## Next Steps

After configuration:

1. [Review API Endpoints](api-endpoints.md) - Understand available endpoints
2. [System Modeling](system-modeling.md) - Learn about architecture
3. [Security Guidelines](authentication-security.md) - Implement security best practices

---

**Last Updated**: April 2026  
**Version**: 1.0.0
