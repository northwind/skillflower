---
name: skillflower
description: Order florist-backed flowers through Skillflower's agent-facing API and CLI.
homepage: https://github.com/northwind/skillflower
metadata:
  openclaw:
    requires:
      anyBins:
        - skillflower
        - npm
      env:
        - SKILLFLOWER_BASE_URL
        - SKILLFLOWER_API_KEY
---

# Skillflower

Use this skill when the user wants to send flowers, manage recipients, plan a
flower-gifting event, recommend bouquets, create checkout sessions, confirm a
tokenized payment, or look up an order.

## Runtime

- Prefer the `skillflower` CLI.
- If the command is unavailable, ask the user to install it with
  `npm install -g @skillflower/cli`.
- Require `SKILLFLOWER_BASE_URL` and `SKILLFLOWER_API_KEY`.
- Use the public Skillflower API through the CLI. Do not call Firestore,
  provider APIs, or private admin routes directly.
- Never fabricate products, checkout sessions, order ids, or provider success.
  Surface the exact CLI/API error and stop.

## Workflow

1. Create or identify the sender.

```bash
skillflower senders create \
  --display-name "Alex" \
  --description "Busy engineer who likes elegant bouquets" \
  --default-currency USD
```

2. Create or identify the receiver.

```bash
skillflower receivers create \
  --sender-id "$SENDER_ID" \
  --display-name "Mom" \
  --description "Retired teacher who likes soft white and pink flowers" \
  --relationship-to-sender mother \
  --recipient-name "Mom" \
  --recipient-phone "+16045550123" \
  --address-line1 "123 Blossom St" \
  --address-line2 "" \
  --city Seattle \
  --state WA \
  --postal-code 98101 \
  --country US \
  --delivery-notes "Call before delivery"
```

3. Create the occasion or recurring event.

```bash
skillflower receiver-events create \
  --sender-id "$SENDER_ID" \
  --receiver-id "$RECEIVER_ID" \
  --event-type birthday \
  --title "Birthday" \
  --description "Yearly birthday bouquet" \
  --date 2026-07-14 \
  --recurrence yearly \
  --tone warm \
  --budget 120 \
  --active true
```

4. Recommend a product.

```bash
skillflower products recommend \
  --sender-id "$SENDER_ID" \
  --receiver-id "$RECEIVER_ID" \
  --event-id "$EVENT_ID" \
  --budget 120 \
  --tone warm
```

5. Create a checkout session using the selected product id.

```bash
skillflower checkout-sessions create \
  --sender-id "$SENDER_ID" \
  --receiver-id "$RECEIVER_ID" \
  --event-id "$EVENT_ID" \
  --selected-product-id "$PRODUCT_ID" \
  --budget 120
```

6. Confirm checkout only with a tokenized payment value.

For local mock-provider testing, use `mock-paid`. For a real provider token,
include customer fields. Never ask for or transmit raw card number, CVV, or
other raw card data through the chat.

```bash
skillflower checkout-sessions confirm \
  --session-id "$SESSION_ID" \
  --payment-token "$PAYMENT_TOKEN" \
  --customer-name "Alex Chen" \
  --customer-email "alex@example.com" \
  --customer-phone "+12065550000" \
  --customer-address1 "500 Sender Ave" \
  --customer-address2 "" \
  --customer-city Seattle \
  --customer-state WA \
  --customer-country US \
  --customer-postal-code 98109 \
  --customer-ip 203.0.113.10
```

7. Look up the order.

```bash
skillflower orders get --order-id "$ORDER_ID"
```

## Response Rules

- Return concise JSON-derived facts: ids, selected product, checkout URL,
  order status, provider order id, and delivery date.
- If payment cannot be completed in the agent runtime, return the checkout
  session id and browser checkout URL for the user to finish.
- If any command exits non-zero, report the command and exact error. Do not
  retry with guessed payloads.
