# API Documentation — Mundaka Surf Academy

This document outlines the API endpoints defined in the `server/` directory. These handlers use the standard `Request` → `Response` API and can run in any standard Node / Serverless / Edge runtime.

---

## 1. Enquiry & Booking Request

- **Endpoint**: `POST /api/enquiry`
- **Purpose**: Handles general contact inquiries or booking requests (when direct online payment is disabled or when requesting reservations).

### Request
- **Headers**:
  - `Content-Type: application/json`
- **Body**:
  ```json
  {
    "type": "booking" | "contact",
    "name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+34600112233",
    "message": "Optional inquiry text",
    "planId": "group",
    "days": 3,
    "people": 2,
    "date": "2026-07-15",
    "time": "10:00",
    "level": "beginner"
  }
  ```

### Response
- **Status**: `200 OK`
- **Body**:
  ```json
  {
    "ok": true,
    "bookingId": "MSA-XXXXXXXX-XXXX",
    "message": "Enquiry received successfully"
  }
  ```

### Error Cases
- **400 Bad Request**:
  ```json
  {
    "error": "Missing required fields: email, name"
  }
  ```
- **500 Internal Server Error**:
  ```json
  {
    "error": "Failed to send email notification"
  }
  ```

---

## 2. Create Checkout Session

- **Endpoint**: `POST /api/create-checkout-session`
- **Purpose**: Validates reservation details, calculates deposit / full charge against `plans.json`, registers a `pending` booking, and generates a Stripe Checkout URL.

### Request
- **Headers**:
  - `Content-Type: application/json`
- **Body**:
  ```json
  {
    "planId": "group",
    "days": 1,
    "people": 2,
    "date": "2026-07-15",
    "time": "10:00",
    "level": "beginner",
    "name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+34600112233",
    "payFull": false
  }
  ```

### Response
- **Status**: `200 OK`
- **Body**:
  ```json
  {
    "url": "https://checkout.stripe.com/c/pay/cs_test_..."
  }
  ```

### Error Cases
- **400 Bad Request**:
  ```json
  {
    "error": "Invalid plan or pricing configuration"
  }
  ```
- **500 Internal Server Error**:
  ```json
  {
    "error": "Stripe session initialization failed"
  }
  ```

---

## 3. Stripe Webhook

- **Endpoint**: `POST /api/stripe-webhook`
- **Purpose**: Verifies Stripe webhook signatures and updates the booking status (`paid`, `failed`, or `cancelled`).

### Request
- **Headers**:
  - `Stripe-Signature: t=...,v1=...`
  - `Content-Type: application/json`
- **Body**: Raw Stripe event payload (`checkout.session.completed`, etc.)

### Response
- **Status**: `200 OK`
- **Body**:
  ```json
  {
    "received": true
  }
  ```

### Error Cases
- **400 Bad Request**:
  ```json
  {
    "error": "Invalid webhook signature"
  }
  ```
- **500 Internal Server Error**:
  ```json
  {
    "error": "Failed to update booking status"
  }
  ```
