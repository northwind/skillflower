---
name: skillflower
description: Let users order florist-backed flowers through the hosted Skillflower service.
homepage: https://github.com/northwind/skillflower
metadata:
  openclaw:
    requires:
      bins:
        - curl
      env:
        - SKILLFLOWER_BASE_URL
        - SKILLFLOWER_API_KEY
---

# Skillflower

Use this skill when the user wants to send flowers, choose a bouquet, create a
checkout session, complete payment, or check an order.

## Requirements

- `SKILLFLOWER_BASE_URL`: hosted Skillflower service URL.
- `SKILLFLOWER_API_KEY`: API key issued for the user or agent.
- `curl`: used to call Skillflower's public API.

If either env var is missing, stop and ask the user to configure it. Do not
invent a host, API key, product, checkout session, or order id.

## User Inputs To Collect

Before ordering, collect:

- sender display name and short preference description
- receiver display name, relationship, phone, and US or Canada delivery address
- occasion title, event type, delivery date, tone, recurrence, and budget
- whether the user wants to continue payment in the browser or already has a
  tokenized payment value

Never ask for raw card number, CVV, or raw bank data in chat. Skillflower only
accepts tokenized payment values or browser checkout handoff.

## API Calls

Use `curl` with:

```bash
-H "Authorization: Bearer $SKILLFLOWER_API_KEY"
-H "Content-Type: application/json"
```

Create sender:

```bash
curl -sS -X POST "$SKILLFLOWER_BASE_URL/api/v1/senders" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"displayName":"Alex","description":"Busy engineer who likes elegant bouquets","defaultCurrency":"USD"}'
```

Create receiver:

```bash
curl -sS -X POST "$SKILLFLOWER_BASE_URL/api/v1/receivers" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","displayName":"Mom","description":"Likes soft white and pink flowers","relationshipToSender":"mother","address":{"recipientName":"Mom","recipientPhone":"+16045550123","line1":"123 Blossom St","line2":"","city":"Seattle","state":"WA","postalCode":"98101","country":"US","deliveryNotes":"Call before delivery"}}'
```

Create event:

```bash
curl -sS -X POST "$SKILLFLOWER_BASE_URL/api/v1/receiver-events" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","receiverId":"RECEIVER_ID","eventType":"birthday","title":"Birthday","description":"Birthday bouquet","date":"2026-07-14","recurrence":"yearly","tone":"warm","budget":120,"active":true}'
```

Recommend product:

```bash
curl -sS -X POST "$SKILLFLOWER_BASE_URL/api/v1/products/recommend" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","receiverId":"RECEIVER_ID","eventId":"EVENT_ID","budget":120,"tone":"warm"}'
```

Create checkout:

```bash
curl -sS -X POST "$SKILLFLOWER_BASE_URL/api/v1/checkout-sessions" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","receiverId":"RECEIVER_ID","eventId":"EVENT_ID","selectedProductId":"PRODUCT_ID","budget":120}'
```

If the user should complete payment in the browser, return:

```text
$SKILLFLOWER_BASE_URL/checkout/SESSION_ID
```

Confirm checkout only when the user has a tokenized payment value:

```bash
curl -sS -X POST "$SKILLFLOWER_BASE_URL/api/v1/checkout-sessions/SESSION_ID/confirm" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"paymentToken":"TOKEN","customer":{"name":"Alex Chen","email":"alex@example.com","phone":"+12065550000","address1":"500 Sender Ave","address2":"","city":"Seattle","state":"WA","country":"US","postalCode":"98109","ip":"203.0.113.10"}}'
```

Get order:

```bash
curl -sS "$SKILLFLOWER_BASE_URL/api/v1/orders/ORDER_ID" \
  -H "Authorization: Bearer $SKILLFLOWER_API_KEY"
```

## Response Rules

- Parse each JSON response and carry forward returned ids.
- Present bouquet name, meaning, price, checkout link, order status, provider
  order id, and delivery date when available.
- If any API call returns an error JSON or non-2xx status, show the exact error
  and stop. Do not retry with guessed fields.
