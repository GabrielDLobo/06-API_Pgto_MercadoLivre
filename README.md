# 💳 Mercado Livre Payment API Integration


### English Version

A Django-based API that integrates with **Mercado Livre's payment system**, allowing secure and automated handling of transactions, order creation, and webhook notifications. This project simulates a real-world e-commerce payment flow with RESTful architecture and external service communication.

### PT/BR
Uma API baseada em Django que integra com o sistema de pagamentos do **Mercado Livre**, permitindo o gerenciamento seguro e automatizado de transações, criação de pedidos e notificações via webhooks. O projeto simula um fluxo de pagamento de e-commerce com arquitetura RESTful e comunicação com serviços externos.

---

## 🚀 Objective | Objetivo

Build an API capable of:

- Creating and managing payment orders via Mercado Livre  
- Handling asynchronous payment notifications through webhooks  
- Simulating real-world e-commerce payment flows  
- Ensuring secure communication and token validation  
- Following RESTful architecture best practices

Criar uma API capaz de:

- Criar e gerenciar pedidos de pagamento via Mercado Livre  
- Lidar com notificações assíncronas de pagamento via webhooks  
- Simular fluxos reais de pagamento em e-commerce  
- Garantir comunicação segura e validação de tokens  
- Seguir boas práticas de arquitetura RESTful

---

## ✨ Features | Funcionalidades

- 🔐 OAuth2 authentication with Mercado Livre  
- 🧾 Order creation and payment tracking  
- 📡 Webhook endpoint for payment status updates  
- 📊 Modular RESTful API structure  
- 🧪 Token validation and error handling  

### PT/BR

- 🔐 Autenticação OAuth2 com o Mercado Livre  
- 🧾 Criação de pedidos e acompanhamento de pagamentos  
- 📡 Endpoint de webhook para atualizações de status de pagamento  
- 📊 Estrutura modular de API RESTful  
- 🧪 Validação de token e tratamento de erros



## ⛏️ Installation / Instalação

Execute o projeto com Python

```bash
  python -m uvicorn app:app --reload
```

---

## 🧪 Tech Stack | Stacks Utilizadas

| Layer         | Technologies                          |
|---------------|----------------------------------------|
| Front-end     | HTML5 and TailwindCSS                  |
| Back-end      | Django Rest Framework, FastAPI, Decouple, UUID and Requests|
| Auth          | OAuth2 (Mercado Livre)                 |
| Webhooks      | Custom Django views + HTTP handlers    |
| External API  | Mercado Livre Payments                 |

---
    
## 📋 Documentação da API - Services

## 🧾 Services Module | Módulo de Serviços

### 🏦 MercadoPago Integration

This API communicates with MercadoPago's payment services to create and manage transactions.  
Esta API se comunica com os serviços de pagamento do MercadoPago para criar e gerenciar transações.

---

### 📡 Endpoints

```http
GET  https://api.mercadopago.com/
POST https://api.mercadopago.com/create_payment
```
GET — Base endpoint for MercadoPago API / Endpoint base da API do MercadoPago

POST — Creates a payment transaction / Cria uma transação de pagamento

---

### 💳 Credit Card Payment | Pagamento com Cartão de Crédito

This endpoint allows you to create a payment using credit card details through MercadoPago.  
Este endpoint permite criar um pagamento usando dados de cartão de crédito via MercadoPago.

#### Request Body | Corpo da Requisição

| Parameter           | Type     | Required | Description / Descrição                     |
|---------------------|----------|----------|---------------------------------------------|
| `card_number`       | number   | ✅       | Credit card number / Número do cartão       |
| `expiration_month`  | integer  | ✅       | Expiration month / Mês de validade          |
| `expiration_year`   | integer  | ✅       | Expiration year / Ano de validade           |
| `security_code`     | number   | ✅       | CVV / Código de segurança                   |
| `cardholder_name`   | string   | ✅       | Name on card / Nome do titular              |
| `amount`            | decimal  | ✅       | Payment amount / Valor da transação         |
| `installments`      | integer  | ❌       | Number of installments / Parcelas           |
| `payment_method_id` | string   | ✅       | Payment method (e.g. visa) / Bandeira do cartão |
| `payer_email`       | string   | ✅       | Customer email / Email do cliente           |


### ✅ Example Payload | Exemplo de Payload

```json
{
  "card_number": "5031755734530604",
  "expiration_month": 11,
  "expiration_year": 2025,
  "security_code": "123",
  "cardholder_name": "Gabriel Lima",
  "amount": 299.90,
  "installments": 1,
  "payment_method_id": "visa",
  "payer_email": "cliente@email.com"
}
```

## 📡 Webhook Response | Resposta via Webhook

Once the payment is processed, MercadoPago sends a webhook to your configured endpoint with the payment status.

Após o processamento do pagamento, o MercadoPago envia um webhook para o endpoint configurado com o status da transação.

Example Webhook Payload | Exemplo de Payload do Webhook
json
{
  "order_id": "MLB123456789",
  "status": "approved",
  "amount": 299.90,
  "payer_email": "cliente@email.com",
  "payment_method": "visa"
}

---

### 🧾 Pix Payment | Pagamento via Pix

This endpoint allows users to make payments using Brazil's instant payment system (Pix).  
Este endpoint permite que usuários realizem pagamentos usando o sistema de pagamento instantâneo do Brasil (Pix).

#### Request Body | Corpo da Requisição

| Parameter | Type   | Required | Description / Descrição         |
|-----------|--------|----------|---------------------------------|
| `email`   | string | ✅       | Customer email / Email do cliente |
| `CPF`     | number | ✅       | Brazilian tax ID / CPF do cliente |


### ✅ Example Payload | Exemplo de Payload

```json
{
  "email": "cliente@email.com",
  "CPF": "12345678900"
}
```

---

### 🧾 Boleto Payment | Pagamento via Boleto Bancário

This endpoint allows users to generate a boleto for offline payment.  
Este endpoint permite gerar um boleto bancário para pagamento offline.

#### Request Body | Corpo da Requisição

| Parameter         | Type     | Required | Description / Descrição                   |
|-------------------|----------|----------|-------------------------------------------|
| `first_name`      | string   | ✅       | First name / Primeiro nome                |
| `last_name`       | string   | ✅       | Last name / Sobrenome                     |
| `email`           | string   | ✅       | Customer email / Email do cliente         |
| `CPF`             | number   | ✅       | Brazilian tax ID / CPF do cliente         |
| `zip_code`        | number   | ✅       | Postal code / CEP                         |
| `street_name`     | string   | ✅       | Street name / Nome da rua                 |
| `street_number`   | number   | ✅       | Street number / Número da residência      |
| `neighborhood`    | string   | ✅       | Neighborhood / Bairro                     |
| `city`            | FK       | ✅       | City reference / Cidade                   |
| `federal_unit`    | string   | ✅       | State abbreviation / Unidade federativa   |


### ✅ Example Payload | Exemplo de Payload

```json
{
  "first_name": "Gabriel",
  "last_name": "Lima",
  "email": "cliente@email.com",
  "CPF": "12345678900",
  "zip_code": "04567-000",
  "street_name": "Rua das Flores",
  "street_number": 123,
  "neighborhood": "Jardins",
  "city": 5,
  "federal_unit": "SP"
}
```

---

## Project Images / Imagens do Projeto

<img width="1480" height="913" alt="image" src="https://github.com/user-attachments/assets/3bbc221f-1733-461e-a98d-bcc8c3269955" />