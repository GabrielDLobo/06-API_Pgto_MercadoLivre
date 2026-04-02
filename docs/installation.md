# Installation Guide

This guide provides step-by-step instructions for installing and setting up the Mercado Libre Payment API on your local development environment.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Running the Application](#running-the-application)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before installing the application, ensure you have the following software and accounts:

### Required Software

| Software | Version | Purpose | Download Link |
|----------|---------|---------|---------------|
| **Python** | 3.8+ | Runtime environment | [python.org](https://www.python.org/downloads/) |
| **pip** | Latest | Package manager | Included with Python |
| **Git** | Latest | Version control | [git-scm.com](https://git-scm.com/) |

### Required Accounts

| Account | Purpose | Link |
|---------|---------|------|
| **Mercado Pago** | Payment processing | [mercadopago.com.br](https://www.mercadopago.com.br/) |
| **Mercado Pago Developer** | API credentials | [developers.mercadopago.com](https://developers.mercadopago.com/) |

### System Requirements

- **Operating System**: Windows 10+, macOS 10.15+, Linux (Ubuntu 18.04+)
- **RAM**: Minimum 2GB (4GB recommended)
- **Disk Space**: 500MB free space
- **Network**: Internet connection for API integration

---

## Installation Steps

### Step 1: Clone the Repository

```bash
# Clone the repository
git clone https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre.git

# Navigate to the project directory
cd 06-API_Pgto_MercadoLivre
```

### Step 2: Create Virtual Environment

It's recommended to use a virtual environment to isolate project dependencies:

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment

# On Windows (Command Prompt)
venv\Scripts\activate

# On Windows (PowerShell)
venv\Scripts\Activate.ps1

# On macOS/Linux
source venv/bin/activate
```

After activation, you should see `(venv)` at the beginning of your terminal prompt.

### Step 3: Install Dependencies

```bash
# Install all required packages
pip install -r requirements.txt
```

This will install the following packages:

| Package | Version | Purpose |
|---------|---------|---------|
| fastapi | 0.115.12 | Web framework |
| uvicorn | 0.34.2 | ASGI server |
| pydantic | 2.11.4 | Data validation |
| requests | 2.32.3 | HTTP client |
| Jinja2 | 3.1.6 | Template engine |
| python-decouple | 3.8 | Configuration management |
| starlette | 0.46.2 | ASGI framework |

### Step 4: Configure Environment Variables

Create a `.env` file in the project root directory:

```bash
# Create .env file
touch .env  # macOS/Linux
# or
echo. > .env  # Windows CMD
```

Add the following configuration:

```env
# Mercado Pago API Configuration
MP_BASE_API_URL=https://api.mercadopago.com
MP_ACCESS_TOKEN=your-access-token-here

# Application Settings
APP_HOST=0.0.0.0
APP_PORT=8000
DEBUG=True
```

> **⚠️ Important**: Never commit the `.env` file to version control. It's already included in `.gitignore`.

---

## Running the Application

### Start the Development Server

```bash
# Run with auto-reload for development
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

### Verify Installation

1. Open your browser and navigate to `http://localhost:8000`
2. You should see the checkout payment page
3. Access API documentation at `http://localhost:8000/docs`

### Expected Output

When the server starts successfully, you should see:

```
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [xxxx]
INFO:     Started server process [xxxx]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

---

## Troubleshooting

### Common Issues

#### 1. Python Version Error

**Error**: `SyntaxError` or `ImportError`

**Solution**: Ensure Python 3.8 or higher is installed:

```bash
python --version  # Should show Python 3.8+
```

#### 2. Module Not Found

**Error**: `ModuleNotFoundError: No module named 'fastapi'`

**Solution**: Activate the virtual environment and reinstall dependencies:

```bash
venv\Scripts\activate  # Windows
source venv/bin/activate  # macOS/Linux
pip install -r requirements.txt
```

#### 3. Port Already in Use

**Error**: `OSError: [Errno 48] Address already in use`

**Solution**: Use a different port:

```bash
uvicorn app:app --reload --port 8001
```

#### 4. Mercado Pago API Connection Error

**Error**: `ConnectionError` or `HTTPError`

**Solution**:
- Verify your internet connection
- Check if `MP_ACCESS_TOKEN` is valid
- Ensure `MP_BASE_API_URL` is correct

#### 5. Virtual Environment Activation Issues

**Windows PowerShell**: If you get an execution policy error:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## Next Steps

After successful installation:

1. [Configure the project](configuration.md) - Set up environment variables and API credentials
2. [Review API endpoints](api-endpoints.md) - Understand available endpoints
3. [Start development](development.md) - Begin building features

---

## Getting Help

If you encounter issues not covered in this guide:

1. Check the [FAQ](#troubleshooting) section
2. Review [Mercado Pago Documentation](https://developers.mercadopago.com/)
3. Open an issue on [GitHub](https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre/issues)

---

**Last Updated**: April 2026  
**Version**: 1.0.0
