# Skillflower Ordering Contract

This is the public contract used by the OpenClaw skill. It is intentionally
limited to the flower-ordering path.

## Configuration

Every request uses:

```http
Authorization: Bearer <SKILLFLOWER_API_KEY>
Content-Type: application/json
```

Base URL:

```text
<SKILLFLOWER_BASE_URL>
```

## Flow

1. Create sender: `POST /api/v1/senders`
2. Create receiver: `POST /api/v1/receivers`
3. Create occasion: `POST /api/v1/receiver-events`
4. Recommend bouquet: `POST /api/v1/products/recommend`
5. Create checkout: `POST /api/v1/checkout-sessions`
6. Complete payment:
   - browser handoff: `/checkout/:sessionId`
   - tokenized confirmation: `POST /api/v1/checkout-sessions/:sessionId/confirm`
7. Check order: `GET /api/v1/orders/:orderId`

## Required User Data

Sender:

- display name
- short preference description

Receiver:

- display name
- relationship to sender
- recipient name and phone
- street, city, state/province, postal code, country `US` or `CA`
- delivery notes, if any

Occasion:

- event type and title
- delivery date
- recurrence: `none`, `yearly`, or `monthly`
- tone
- budget in USD

Payment:

- browser checkout is the default
- tokenized payment value only when provided by an approved payment flow

## Agent Rules

- Carry ids forward from API responses.
- Use `primary.id` from product recommendation unless the user chooses another option.
- Construct browser checkout URL as `<SKILLFLOWER_BASE_URL>/checkout/<sessionId>`.
- Do not collect raw card number, CVV, or bank data.
- Do not invent products, checkout sessions, provider order ids, payment success, or order status.

## Errors

Error responses are JSON:

```json
{
  "error": "Receiver does not belong to sender"
}
```

If an error response appears, report the exact error and stop.
