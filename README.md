# 💻📱 API Pgto Mercado Livre

Construção de uma API de pagamentos do Mercado Livre utilizando tecnologias de última geração: HTML5, TailwindCSS, Django e FASTAPI.


## ⛏️ Instalação

Execute o projeto com Python

```bash
  python -m uvicorn app:app --reload
```
    
## 💎 Stacks utilizadas

**Front-end:** HTML5 e TailwindCSS

**Back-end:** Django, FASTAPI, Decouple, UUID e Requests

## 📋 Documentação da API - Services

  #### Services - MercadoPago
  
  ```http
  GET  https://api.mercadopago.com/
  POST https://api.mercadopago.com/create_payment
```

#### Se cartão de crédito

| Parâmetro   | Tipo       | Descrição                           |
| :---------- | :--------- | :---------------------------------- |
| `card_number` | `number` | **Obrigatório** |
| `expiration_month` | `datetime` | **Obrigatório** |
| `expiration_year` | `datetime` | **Obrigatório** |
| `security_code` | `number` | **Obrigatório** |
| `cardholder name` | `string` | **Obrigatório** |
| `cardholder CPF` | `number` | **Obrigatório** |

#### Se pix

| Parâmetro   | Tipo       | Descrição                           |
| :---------- | :--------- | :---------------------------------- |
| `email` | `string` | **Obrigatório** |
| `CPF` | `number` | **Obrigatório** |

#### Se boleto

| Parâmetro   | Tipo       | Descrição                           |
| :---------- | :--------- | :---------------------------------- |
| `first name` | `string` | **Obrigatório** |
| `last name` | `string` | **Obrigatório** |
| `email` | `string` | **Obrigatório** |
| `CPF` | `number` | **Obrigatório** |
| `Zip Code` | `number` | **Obrigatório** |
| `street name` | `string` | **Obrigatório** |
| `street number` | `number` | **Obrigatório** |
| `neighborhood` | `string` | **Obrigatório** |
| `city` | `fk` | **Obrigatório** |
| `federal_unit` | `string` | **Obrigatório** |
