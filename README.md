# Mercado Pago Payment API - FastAPI

A FastAPI-based payment integration API connected to Mercado Pago for secure transaction processing, payment checkout creation, and webhook notifications.

## Documentation

Full project documentation is available at:
<a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/" target="_blank" rel="noopener noreferrer">https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/</a>

### Documentation Sections

- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/" target="_blank" rel="noopener noreferrer">Home</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/installation/" target="_blank" rel="noopener noreferrer">Installation</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/configuration/" target="_blank" rel="noopener noreferrer">Configuration</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/guidelines/" target="_blank" rel="noopener noreferrer">Guidelines and Standards</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/project-structure/" target="_blank" rel="noopener noreferrer">Project Structure</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/api-endpoints/" target="_blank" rel="noopener noreferrer">API Endpoints</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/system-modeling/" target="_blank" rel="noopener noreferrer">System Modeling</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/authentication-security/" target="_blank" rel="noopener noreferrer">Authentication and Security</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/development/" target="_blank" rel="noopener noreferrer">Development</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/testing/" target="_blank" rel="noopener noreferrer">Testing</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/deploy/" target="_blank" rel="noopener noreferrer">Deploy</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/contributing/" target="_blank" rel="noopener noreferrer">Contributing</a>
- <a href="https://gabrieldlobo.github.io/06-API_Pgto_MercadoLivre/release-notes/" target="_blank" rel="noopener noreferrer">Release Notes</a>

### Local preview

```bash
mkdocs serve -a 127.0.0.1:8001
```

Open:
<a href="http://127.0.0.1:8001/" target="_blank" rel="noopener noreferrer">http://127.0.0.1:8001/</a>

### Docs source

Edit markdown pages in `docs/` and navigation in `mkdocs.yml`.

### Publish

```bash
mkdocs gh-deploy --clean
```

## Key Features

- Mercado Pago integration for secure payments
- Credit card, PIX, and boleto support
- Payment checkout creation endpoint
- Webhook endpoint for asynchronous notifications
- Real-time payment status retrieval
- RESTful API architecture with request validation
- HTML checkout template for flow visualization

## Tech Stack

- Python / FastAPI
- Uvicorn
- Pydantic
- Requests
- Jinja2 templates
- MkDocs + Material for project documentation

## Project Structure (high-level)

Main modules in this repository:

- `app.py` - FastAPI routes and request handling
- `services/mercadopago.py` - Mercado Pago integration service
- `templates/checkout.html` - checkout page template
- `docs/` - full documentation source

## Main Routes

- `GET /` - Checkout page
- `POST /create_payment` - Create a payment
- `POST /webhook` - Receive asynchronous Mercado Pago notifications
- `GET /payment/{payment_id}` - Retrieve payment details

## Getting Started (development)

### 1) Clone and use the existing virtual environment

```bash
git clone https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre.git
cd 06-API_Pgto_MercadoLivre

# Windows PowerShell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\venv\Scripts\Activate.ps1
```

### 2) Install dependencies

```bash
pip install -r requirements.txt
```

### 3) Configure environment variables

Create a `.env` file in the project root with your Mercado Pago credentials.

### 4) Run the API

```bash
python -m uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Open:

- http://127.0.0.1:8000/
- http://127.0.0.1:8000/docs

### 5) Run documentation locally

```bash
mkdocs serve -a 127.0.0.1:8001
```

Open:

- http://127.0.0.1:8001/

## License

No license file is included by default. Add one if you plan to distribute this project.
