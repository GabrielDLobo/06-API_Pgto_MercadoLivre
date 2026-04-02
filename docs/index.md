# Mercado Pago Payment API

Documentation for the FastAPI payment integration with Mercado Pago.

<div class="page-hero">
   <div class="hero-copy">
      <p class="hero-eyebrow">FastAPI + Mercado Pago</p>
      <h1>Pagamentos online com fluxo claro, seguro e rápido</h1>
      <p>
         Documentação central do projeto de pagamentos com integração ao Mercado Pago.
         Aqui você encontra instalação, configuração, endpoints, segurança, testes e deploy em um só lugar.
      </p>
      <div class="hero-actions">
         <a class="md-button md-button--primary" href="installation.md">Começar agora</a>
         <a class="md-button" href="api-endpoints.md">Ver endpoints</a>
         <a class="md-button" href="system-modeling.md">Arquitetura</a>
      </div>
   </div>
   <div class="hero-panel">
      <div class="hero-stat">
         <span class="hero-stat__value">3</span>
         <span class="hero-stat__label">métodos de pagamento</span>
      </div>
      <div class="hero-stat">
         <span class="hero-stat__value">1</span>
         <span class="hero-stat__label">API com foco em checkout</span>
      </div>
      <div class="hero-stat">
         <span class="hero-stat__value">100%</span>
         <span class="hero-stat__label">documentação navegável</span>
      </div>
   </div>
</div>

## Visão Geral

<div class="grid cards metric-grid" markdown>

- :material-credit-card-check-outline: **Checkout unificado**

    Cartão, PIX e boleto com fluxo consistente para o front-end.

- :material-shield-check-outline: **Segurança aplicada**

    Token, validação de entrada e tratamento de erros na integração.

- :material-flash-outline: **Resposta em tempo real**

    Retorno rápido para confirmar o estado da transação.

- :material-api: **Integração REST organizada**

    Endpoints separados para checkout, consulta e operação.

</div>

## Documentation Index

<div class="grid cards" markdown>

- **Getting Started**

   Install the project, configure your environment, and run the API locally.

   [Installation](installation.md)  
   [Configuration](configuration.md)  
   [Development](development.md)

- **API & Security**

   Review the request flow, endpoint reference, and security guidance.

   [API Endpoints](api-endpoints.md)  
   [Authentication and Security](authentication-security.md)  
   [System Modeling](system-modeling.md)

- **Quality & Delivery**

   Standards, testing, deployment, and release notes.

   [Guidelines and Standards](guidelines.md)  
   [Testing](testing.md)  
   [Deploy](deploy.md)  
   [Contributing](contributing.md)  
   [Release Notes](release-notes.md)

</div>

## Quick Start

```bash
git clone https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre.git
cd 06-API_Pgto_MercadoLivre
pip install -r requirements.txt
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

::: details Interactive API docs

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

:::

## Navegue pela documentação

<div class="grid cards doc-grid" markdown>

- **Instalação e configuração**

   [Installation](installation.md)  
   [Configuration](configuration.md)  
   [Development](development.md)

- **Arquitetura e API**

   [API Endpoints](api-endpoints.md)  
   [Authentication and Security](authentication-security.md)  
   [System Modeling](system-modeling.md)

- **Boas práticas e operação**

   [Guidelines and Standards](guidelines.md)  
   [Testing](testing.md)  
   [Deploy](deploy.md)

- **Contribuição e histórico**

   [Contributing](contributing.md)  
   [Release Notes](release-notes.md)

</div>

## Payment Flow

```mermaid
sequenceDiagram
      participant User
      participant Frontend
      participant API
      participant MercadoPago

      User->>Frontend: Escolhe metodo de pagamento
      Frontend->>API: POST /create_payment
      API->>MercadoPago: Cria cobranca
      MercadoPago-->>API: Retorna status
      API-->>Frontend: Resultado da transacao
      Frontend->>User: Exibe confirmacao
```

   ## Tech Stack

| Camada | Tecnologia |
|---|---|
| Backend | FastAPI 0.115.12 |
| Servidor | Uvicorn 0.34.2 |
| Validacao | Pydantic 2.11.4 |
| Cliente HTTP | Requests 2.32.3 |
| Templates | Jinja2 3.1.6 |
| Configuracao | python-decouple 3.8 |

## Atalhos Úteis

<div class="highlight-box">
<strong>Repositório:</strong> <a href="https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre" target="_blank" rel="noopener noreferrer">06-API_Pgto_MercadoLivre</a><br>
<strong>Docs locais:</strong> `mkdocs serve -a 127.0.0.1:8001`<br>
<strong>API local:</strong> `uvicorn app:app --reload --host 0.0.0.0 --port 8000`
</div>

## Repository

<a href="https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre" target="_blank" rel="noopener noreferrer">06-API_Pgto_MercadoLivre</a>
