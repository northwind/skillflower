# Skillflower API

## Overview

Skillflower exposes a versioned public API under `/api/v1` for Agent-driven flower gifting.

The canonical public resources are:

- `senders`
- `receivers`
- `receiver-events`
- `products/recommend`
- `checkout-sessions`
- `orders`

The older `POST /api/skill` route remains available for legacy compatibility, but new integrations should target `/api/v1`. Legacy requests use the same bearer-token authentication as the public API.

## Authentication

Send every public API request with:

```http
Authorization: Bearer YOUR_SKILLFLOWER_API_KEY
```

Authentication semantics:

- `401` missing bearer token
- `403` invalid bearer token

## Resource Model

Skillflower uses an explicit gifting graph:

- A `sender` represents the gifting user or account.
- A `receiver` belongs to exactly one sender.
- A `receiver-event` belongs to exactly one sender and one receiver.
- A `checkout-session` references one sender, one receiver, one event, and one selected product.
- An `order` is produced from a confirmed checkout session.

Ownership is enforced on recommendation and checkout endpoints. If a receiver or event does not belong to the supplied sender, the API returns `400`.

## AI-Enriched Descriptions

Both `sender.description` and `receiver.description` are first-class fields.

They are meant to accept a short natural-language description such as:

```json
{
  "description": "Retired teacher who likes soft pink flowers and prefers thoughtful, not flashy gifts"
}
```

Skillflower uses Gemini to enrich structured fields such as profile notes, preferences, occupation, gender, locale, timezone, or age range when enough signal is present.

If upstream AI enrichment fails, the API surfaces the failure explicitly instead of silently generating guessed content.

## Senders

### Create sender

`POST /api/v1/senders`

```json
{
  "displayName": "Alex",
  "description": "Busy engineer who likes elegant bouquets",
  "defaultCurrency": "USD"
}
```

### Get sender

`GET /api/v1/senders/:senderId`

### Update sender

`PATCH /api/v1/senders/:senderId`

```json
{
  "description": "Cross-border engineer who sends flowers often"
}
```

### Delete sender

`DELETE /api/v1/senders/:senderId`

## Receivers

### Create receiver

`POST /api/v1/receivers`

```json
{
  "senderId": "sender-1",
  "displayName": "Mom",
  "description": "Retired teacher who likes soft white and pink flowers",
  "relationshipToSender": "mother",
  "address": {
    "recipientName": "Mom",
    "recipientPhone": "+16045550123",
    "line1": "123 Blossom St",
    "line2": "",
    "city": "Vancouver",
    "state": "BC",
    "postalCode": "V5K0A1",
    "country": "CA",
    "deliveryNotes": "Call before delivery"
  }
}
```

### List receivers by sender

`GET /api/v1/receivers?senderId=sender-1`

### Get receiver

`GET /api/v1/receivers/:receiverId`

### Update receiver

`PATCH /api/v1/receivers/:receiverId`

Receiver updates support partial address patches. For example:

```json
{
  "address": {
    "city": "Seattle",
    "deliveryNotes": "Leave with concierge"
  }
}
```

### Delete receiver

`DELETE /api/v1/receivers/:receiverId`

## Receiver Events

### Create receiver event

`POST /api/v1/receiver-events`

```json
{
  "senderId": "sender-1",
  "receiverId": "receiver-1",
  "eventType": "birthday",
  "title": "Birthday",
  "description": "Yearly birthday bouquet",
  "date": "2026-06-01",
  "recurrence": "yearly",
  "tone": "warm",
  "budget": 120,
  "active": true
}
```

### List receiver events

`GET /api/v1/receiver-events?receiverId=receiver-1`

### Get receiver event

`GET /api/v1/receiver-events/:eventId`

### Update receiver event

`PATCH /api/v1/receiver-events/:eventId`

### Delete receiver event

`DELETE /api/v1/receiver-events/:eventId`

## Product Recommendation

### Recommend products

`POST /api/v1/products/recommend`

```json
{
  "senderId": "sender-1",
  "receiverId": "receiver-1",
  "eventId": "event-1",
  "budget": 120,
  "tone": "warm"
}
```

Example response:

```json
{
  "primary": {
    "id": "florist-one-pink-elegance",
    "name": "Fresh",
    "imageUrl": "https://cdn.floristone.com/small/FAA-103.jpg",
    "meaning": "Graceful affection, warmth, and romantic care.",
    "description": "A polished pink arrangement suitable for affectionate milestone gifts.",
    "category": "romance",
    "price": 74.95,
    "currency": "USD",
    "sourceProductId": "FAA-103",
    "sourceProvider": "florist_one"
  },
  "backups": [
    {
      "id": "florist-one-classic-mixed-bouquet",
      "name": "Classic Mixed Bouquet",
      "imageUrl": "https://www.floristone.com/flowers/thumbnails/VA00101S.jpg",
      "meaning": "Simple appreciation and dependable thoughtfulness."
    }
  ],
  "reasoningSummary": "Recommended Pink Elegance for a mother birthday gift within a 120 USD budget.",
  "priceRange": {
    "min": 55,
    "max": 120
  },
  "currency": "USD"
}
```

## Checkout Sessions

### Create checkout session

`POST /api/v1/checkout-sessions`

```json
{
  "senderId": "sender-1",
  "receiverId": "receiver-1",
  "eventId": "event-1",
  "selectedProductId": "florist-one-pink-elegance",
  "budget": 120
}
```

Example response:

```json
{
  "id": "session-1",
  "senderId": "sender-1",
  "receiverId": "receiver-1",
  "eventId": "event-1",
  "selectedProduct": {
    "id": "florist-one-pink-elegance",
    "name": "Pink Elegance",
    "imageUrl": "https://example.com/pink-elegance.jpg",
    "meaning": "Warm affection and grace"
  },
  "budget": 120,
  "currency": "USD",
  "status": "awaiting_payment"
}
```

### Get checkout session

`GET /api/v1/checkout-sessions/:sessionId`

### Confirm checkout session

`POST /api/v1/checkout-sessions/:sessionId/confirm`

```json
{
  "paymentToken": "provider-token",
  "customer": {
    "name": "Alex Chen",
    "email": "alex@example.com",
    "phone": "+12065550000",
    "address1": "500 Sender Ave",
    "address2": "Apt 8",
    "city": "Seattle",
    "state": "WA",
    "country": "US",
    "postalCode": "98109",
    "ip": "203.0.113.10"
  }
}
```

Example response:

```json
{
  "id": "public-order-session-1",
  "senderId": "sender-1",
  "receiverId": "receiver-1",
  "eventId": "event-1",
  "checkoutSessionId": "session-1",
  "floristOneOrderId": "FO-123",
  "bouquetName": "Pink Elegance",
  "deliveryDate": "2026-06-01",
  "status": "confirmed"
}
```

Confirmation semantics:

- The API marks the session as paid, submits the florist order, and creates a public order.
- If the session is already confirmed, the endpoint returns the existing order instead of placing a duplicate florist order.
- Local development with the placeholder `PAYMENT_PROVIDER_*` values from `.env` is strict mock mode, so confirmation must use `paymentToken=mock-paid`.
- Real provider tokens must include `customer`; local mock mode fills a test customer only for `mock-paid`.
- Local provider simulation is explicit: start `npm run mock:florist-one` and launch the app with `FLORIST_ONE_BASE_URL=http://127.0.0.1:31337`.
- Non-mock tokens in local mock mode return `400`.
- Florist One placement failures are explicit failures and do not create confirmed public orders.
- Checkout session states are `draft`, `awaiting_payment`, `paid`, `florist_submitted`, `confirmed`, `florist_failed`, and `expired`.

Browser handoff:

- `/checkout/:sessionId` renders the public checkout session, selected product summary, sender/receiver/event identity, local mock-mode guidance, and success/error states.
- The handoff page does not collect raw card data. Today it only renders the manual token form for explicit local mock mode.
- Live checkout intentionally blocks the manual token form until Florist One's tokenized payment UI contract is wired.
- Form confirmation redirects back to `/checkout/:sessionId?status=success&orderId=...` or `/checkout/:sessionId?status=error&message=...`.
- Programmatic Agents should prefer `/api/v1/checkout-sessions/:sessionId/confirm` with Bearer auth.

## Orders

### List orders by sender

`GET /api/v1/orders?senderId=sender-1`

### Get order

`GET /api/v1/orders/:orderId`

Example not-found response:

```json
{
  "error": "Order not found"
}
```

## Error Semantics

Common response patterns:

- `400` invalid payload, missing query parameter, or invalid resource ownership graph
- `401` missing API authentication
- `403` invalid API authentication
- `404` missing sender, receiver, receiver event, checkout session, selected product, or order
- `5xx` upstream florist failure, upstream AI enrichment failure, or internal service failure

Error bodies are JSON and currently follow the simple shape:

```json
{
  "error": "Receiver does not belong to sender"
}
```
