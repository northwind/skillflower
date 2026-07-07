# Skillflower Ordering Contract

This document describes the public ordering path used by the OpenClaw skill.

## Required Configuration

- `SKILLFLOWER_BASE_URL`: hosted Skillflower service URL.
- `SKILLFLOWER_API_KEY`: bearer token for the user or agent.

Every API request uses:

```http
Authorization: Bearer <SKILLFLOWER_API_KEY>
Content-Type: application/json
```

## Public Workflow

1. `POST /api/v1/senders`
2. `POST /api/v1/receivers`
3. `POST /api/v1/receiver-events`
4. `POST /api/v1/products/recommend`
5. `POST /api/v1/checkout-sessions`
6. Browser payment at `/checkout/:sessionId` or tokenized confirmation at
   `POST /api/v1/checkout-sessions/:sessionId/confirm`
7. `GET /api/v1/orders/:orderId`

## Data The Agent Must Collect

- sender display name and short preference description
- receiver name, relationship, phone, and US or Canada delivery address
- occasion type, title, delivery date, recurrence, tone, and budget
- selected bouquet from the recommendation response
- browser checkout preference or tokenized payment value

## Error Contract

Skillflower returns explicit JSON errors:

```json
{
  "error": "Receiver does not belong to sender"
}
```

Agents must report the exact error and stop. They must not fabricate products,
checkout sessions, provider order ids, payment success, or order status.

## Payment Contract

Skillflower does not accept raw card data in chat. Use the browser checkout URL
or a tokenized payment value supplied by an approved payment flow.
