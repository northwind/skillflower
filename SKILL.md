---
name: send-flowers
description: Help users order florist-backed flowers through the hosted Send Flowers service.
homepage: https://github.com/northwind/skillflower
metadata:
  openclaw:
    requires:
      bins:
        - curl
---

# Send Flowers

Use this skill when a user wants to send flowers, choose a bouquet, create
checkout, complete payment, or check an order.

## Configuration Check

Use the default hosted service URL unless the user overrides it:

```bash
BASE_URL="${SEND_FLOWERS_BASE_URL:-https://skillflower.goosepod.org}"
```

Before calling the API, load the API key from the environment or local cache:

```bash
if [ -z "$SEND_FLOWERS_API_KEY" ] && [ -f "$HOME/.send-flowers/api-key" ]; then
  export SEND_FLOWERS_API_KEY="$(cat "$HOME/.send-flowers/api-key")"
fi
```

If `SEND_FLOWERS_API_KEY` is still missing, stop and ask the user for a
Send Flowers API key. If the user provides one, cache it only after they
agree:

```bash
mkdir -p "$HOME/.send-flowers"
printf '%s\n' "$SEND_FLOWERS_API_KEY" > "$HOME/.send-flowers/api-key"
chmod 600 "$HOME/.send-flowers/api-key"
```

Do not generate a random local API key. The current hosted service accepts only
server-issued keys. Do not invent a host, API key, product, checkout session,
provider order id, or successful order.

## Information To Collect

Collect only what is needed to place the order:

- sender name and a short preference description
- receiver name, relationship, phone, and US or Canada delivery address
- occasion type, title, delivery date, recurrence, tone, and budget
- whether the user wants browser checkout or has an approved tokenized payment value

Never ask for raw card number, CVV, or bank data in chat. The default public
flow is browser checkout.

## First-Time Receiver Check

Before creating a receiver, if the user has not already provided recipient
delivery details, stop and ask for:

- recipient delivery name and phone
- street address, city, state or province, postal code, and country
- optional apartment, suite, delivery notes, and relationship to sender

Do not use the sample receiver address for a real user. If the user gives only
partial details, ask only for the missing required delivery fields.

## API Helper

Use the hosted public API with bearer auth:

```bash
curl -sS -X POST "$BASE_URL/api/v1/..." \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

If a response contains an `error` field, report that exact error and stop.

## Ordering Flow

1. Create sender.

```bash
curl -sS -X POST "$BASE_URL/api/v1/senders" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"displayName":"Alex","description":"Busy engineer who likes elegant bouquets","defaultCurrency":"USD"}'
```

2. Create receiver.

```bash
curl -sS -X POST "$BASE_URL/api/v1/receivers" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","displayName":"Mom","description":"Likes soft white and pink flowers","relationshipToSender":"mother","address":{"recipientName":"Mom","recipientPhone":"+16045550123","line1":"123 Blossom St","line2":"","city":"Seattle","state":"WA","postalCode":"98101","country":"US","deliveryNotes":"Call before delivery"}}'
```

3. Create occasion.

```bash
curl -sS -X POST "$BASE_URL/api/v1/receiver-events" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","receiverId":"RECEIVER_ID","eventType":"birthday","title":"Birthday","description":"Birthday bouquet","date":"2026-07-14","recurrence":"yearly","tone":"warm","budget":120,"active":true}'
```

4. Recommend bouquet.

```bash
curl -sS -X POST "$BASE_URL/api/v1/products/recommend" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","receiverId":"RECEIVER_ID","eventId":"EVENT_ID","budget":120,"tone":"warm"}'
```

Use the returned `primary.id` unless the user chooses a backup option.

5. Create checkout.

```bash
curl -sS -X POST "$BASE_URL/api/v1/checkout-sessions" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"senderId":"SENDER_ID","receiverId":"RECEIVER_ID","eventId":"EVENT_ID","selectedProductId":"PRODUCT_ID","budget":120}'
```

For normal users, return the browser checkout link:

```text
$BASE_URL/checkout/SESSION_ID
```

6. Confirm checkout only if the user already has a tokenized payment value from
an approved payment flow.

```bash
curl -sS -X POST "$BASE_URL/api/v1/checkout-sessions/SESSION_ID/confirm" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"paymentToken":"TOKEN","customer":{"name":"Alex Chen","email":"alex@example.com","phone":"+12065550000","address1":"500 Sender Ave","address2":"","city":"Seattle","state":"WA","country":"US","postalCode":"98109","ip":"203.0.113.10"}}'
```

7. Look up order.

```bash
curl -sS "$BASE_URL/api/v1/orders/ORDER_ID" \
  -H "Authorization: Bearer $SEND_FLOWERS_API_KEY"
```

## User Response

Return concise facts from JSON:

- selected bouquet name, meaning, and price
- checkout link
- order id and provider order id
- order status and delivery date

If payment is not complete, make that explicit and return the checkout link.
