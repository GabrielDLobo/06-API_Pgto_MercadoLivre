# 06-API_Pgto_MercadoLivre

A FastAPI-based payment integration API that connects with Mercado Pago (Mercado Livre's payment system), enabling secure and automated handling of transactions, order creation, and webhook notifications.  This project simulates a real-world e-commerce payment flow with RESTful architecture and external service communication. 

## Main Features

- Integration with Mercado Pago payment gateway
- Secure payment processing and transaction management
- Order creation and tracking
- Webhook endpoint for payment notifications
- Real-time payment status updates
- RESTful API architecture
- HTML templates for payment flow visualization
- Environment-based configuration for security
- Request validation using Pydantic models

---

## Table of Contents

- [Technologies](#technologies)
- [Requirements](#requirements)
- [Setup Instructions](#setup-instructions)
- [Running Locally](#running-locally)
- [API Endpoints](#api-endpoints)
- [Configuration](#configuration)
- [License](#license)

---

## Technologies

This project leverages the following key technologies:

- **Backend**: 
  - FastAPI 0.115.12
  - Uvicorn 0.34.2
  - Python 3.8+
  - Starlette 0.46.2
  
- **Data Validation**:
  - Pydantic 2.11.4
  - Pydantic Core 2.33.2
  
- **HTTP Client**:
  - Requests 2.32.3
  
- **Template Engine**:
  - Jinja2 3.1.6
  
- **Configuration**:
  - python-decouple 3.8
  
- **Additional Libraries**:
  - certifi 2025.4.26
  - urllib3 2.4.0

---

## Requirements

Make sure you have the following installed: 

- Python 3.8+
- pip (Python package manager)
- Mercado Pago account with API credentials
- ngrok or similar tool for webhook testing (development)

---

## Setup Instructions

### Clone the Repository

```bash
git clone https://github.com/GabrielDLobo/06-API_Pgto_MercadoLivre.git
cd 06-API_Pgto_MercadoLivre
```

### Install Dependencies

```bash
pip install -r requirements. txt
```

### Configure Environment Variables

Create a `.env` file in the project root: 

```env
# Mercado Pago Credentials
MERCADO_PAGO_ACCESS_TOKEN=your-access-token-here
MERCADO_PAGO_PUBLIC_KEY=your-public-key-here

# Application Settings
APP_HOST=0.0.0.0
APP_PORT=8000
DEBUG=True

# Webhook Configuration
WEBHOOK_URL=https://your-domain.com/webhook
NOTIFICATION_URL=https://your-domain.com/notifications

# Security
SECRET_KEY=your-secret-key-here
```

### Obtain Mercado Pago Credentials

1. Create a Mercado Pago developer account at [developers.mercadopago.com](https://developers.mercadopago.com)
2. Create a new application
3. Copy your Access Token and Public Key
4. Add them to your `.env` file

---

## Running Locally

### Start the Application

```bash
uvicorn app: app --reload --host 0.0.0.0 --port 8000
```

The API will be available at `http://localhost:8000`.

### API Documentation

Access the interactive API documentation: 

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

---

## API Endpoints

### Payment Endpoints

#### Create Payment Preference

```
POST /create-preference
```

Creates a payment preference in Mercado Pago. 

**Request Body:**
```json
{
  "title": "Product Name",
  "quantity": 1,
  "unit_price": 100.00,
  "description": "Product description"
}
```

**Response:**
```json
{
  "preference_id": "123456789",
  "init_point": "https://www.mercadopago.com. br/checkout/v1/redirect? pref_id=123456789"
}
```

#### Payment Notification Webhook

```
POST /webhook
```

Receives payment notifications from Mercado Pago.

**Headers:**
```
x-signature: <signature>
x-request-id: <request-id>
```

#### Get Payment Status

```
GET /payment/{payment_id}
```

Retrieves the current status of a payment.

**Response:**
```json
{
  "id": "123456789",
  "status": "approved",
  "status_detail": "accredited",
  "transaction_amount": 100.00
}
```

---

## Configuration

### Webhook Setup

For development, use ngrok to expose your local server:

```bash
ngrok http 8000
```

Copy the generated URL and configure it in your Mercado Pago application settings.

### Payment Flow

1. User initiates a purchase on your application
2. Application calls `/create-preference` endpoint
3. User is redirected to Mercado Pago checkout
4. User completes payment
5. Mercado Pago sends notification to `/webhook` endpoint
6. Application processes payment status
7. User is redirected back to success/failure page

---

## Project Structure

```
.
|-- app. py                  # Main FastAPI application
|-- requirements.txt        # Python dependencies
|-- services/               # Business logic and external services
|-- templates/              # HTML templates for payment pages
```

---

## Payment Status Types

The API handles the following payment statuses:

- `pending` - Payment is being processed
- `approved` - Payment was approved
- `authorized` - Payment was authorized but not captured
- `in_process` - Payment is under review
- `in_mediation` - User initiated a dispute
- `rejected` - Payment was rejected
- `cancelled` - Payment was cancelled
- `refunded` - Payment was refunded
- `charged_back` - Payment was charged back

---

## Security Considerations

### Best Practices

- Never expose your Access Token in client-side code
- Store credentials in environment variables
- Validate webhook signatures to ensure authenticity
- Use HTTPS in production
- Implement rate limiting
- Log all transactions for audit purposes
- Validate all incoming payment data

### Webhook Signature Validation

Mercado Pago sends a signature with each webhook request.  Always validate this signature before processing the notification.

---

## Testing

### Test Cards (Sandbox Mode)

Mercado Pago provides test cards for development: 

**Approved:**
- Card:  5031 7557 3453 0604
- CVV: 123
- Expiration: 11/25

**Rejected:**
- Card: 5031 4332 1540 6351
- CVV: 123
- Expiration: 11/25

### Running Tests

```bash
# Create a test payment
curl -X POST http://localhost:8000/create-preference \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test Product",
    "quantity": 1,
    "unit_price":  10.00
  }'
```

---

## Deployment

### Production Deployment

Recommended platforms: 

- **Render** - [render.com](https://render.com)
- **Railway** - [railway.app](https://railway.app)
- **Heroku** - [heroku.com](https://heroku.com)
- **AWS Elastic Beanstalk**
- **DigitalOcean App Platform**

### Environment Configuration

Ensure the following for production:

- Set `DEBUG=False`
- Use production Mercado Pago credentials
- Configure a valid SSL certificate (HTTPS)
- Set up proper logging and monitoring
- Configure webhook URL with your production domain

---

## Error Handling

The API implements comprehensive error handling:

- `400 Bad Request` - Invalid payment data
- `401 Unauthorized` - Invalid credentials
- `404 Not Found` - Payment not found
- `422 Unprocessable Entity` - Validation error
- `500 Internal Server Error` - Server error

---

## Monitoring and Logging

Implement logging for: 

- All payment creation requests
- Webhook notifications received
- Payment status changes
- API errors and exceptions
- Transaction audit trail

---

## License

This project is open source, but no license was explicitly defined. 

Feel free to clone, use, and contribute back! 

---

## Contributing

Pull requests are welcome!  For large changes, consider opening an issue first to discuss what you would like to change.

---

## Resources

- [Mercado Pago API Documentation](https://www.mercadopago.com. br/developers)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Webhook Integration Guide](https://www.mercadopago.com. br/developers/en/guides/notifications/webhooks)

---

## Project Images / Imagens do Projeto

<img width="1480" height="913" alt="image" src="https://github.com/user-attachments/assets/3bbc221f-1733-461e-a98d-bcc8c3269955" />