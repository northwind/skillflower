# Skillflower CLI

## Overview

`@skillflower/cli` is the command-line client for the public Skillflower API.

It is designed for Agent runtimes, scripts, and shell automations that want JSON-first access to the same `/api/v1` resource model documented in [`API.md`](API.md).

## Install

Global install:

```bash
npm install -g @skillflower/cli
```

Local packaging from this repo:

```bash
npm pack --workspace @skillflower/cli
```

Run the local workspace CLI without publishing:

```bash
npm exec --workspace @skillflower/cli -- skillflower --help
```

Note the required `--` separator before `skillflower`. Without it, `npm exec` may consume CLI flags like `--display-name` itself instead of passing them to Skillflower.

## Authentication

Set the target host and public API key:

```bash
export SKILLFLOWER_BASE_URL="https://your-skillflower-host"
export SKILLFLOWER_API_KEY="your-public-api-key"
```

The CLI sends those values as:

- Base URL for the target Skillflower deployment
- `Authorization: Bearer <apiKey>` for every request

For local development against this repo's default `apphosting.yaml`, the checkout confirmation endpoints run in explicit mock-payment mode. Confirm checkout sessions with `paymentToken=mock-paid`.

## Output and Errors

The CLI prints JSON to stdout by default so Agents can consume it directly.

If the API returns an error:

- the CLI exits non-zero
- the CLI prints the server error message to stderr

## Commands

### Minimum external Agent workflow

```bash
skillflower senders create \
  --display-name "Alex" \
  --description "Busy engineer who likes elegant bouquets" \
  --default-currency USD

skillflower receivers create \
  --sender-id sender-1 \
  --display-name "Mom" \
  --description "Retired teacher who likes light colors" \
  --relationship-to-sender mother \
  --recipient-name "Mom" \
  --recipient-phone "+16045550123" \
  --address-line1 "123 Blossom St" \
  --address-line2 "" \
  --city Vancouver \
  --state BC \
  --postal-code V5K0A1 \
  --country CA \
  --delivery-notes ""

skillflower receiver-events create \
  --sender-id sender-1 \
  --receiver-id receiver-1 \
  --event-type birthday \
  --title "Birthday" \
  --description "Yearly birthday bouquet" \
  --date 2026-06-01 \
  --recurrence yearly \
  --tone warm \
  --budget 120 \
  --active true

skillflower products recommend \
  --sender-id sender-1 \
  --receiver-id receiver-1 \
  --event-id event-1 \
  --budget 120 \
  --tone warm

skillflower checkout-sessions create \
  --sender-id sender-1 \
  --receiver-id receiver-1 \
  --event-id event-1 \
  --selected-product-id florist-one-pink-elegance \
  --budget 120

skillflower checkout-sessions confirm \
  --session-id session-1 \
  --payment-token mock-paid

skillflower orders get --order-id public-order-session-1
```

When confirming with a real Florist One sandbox token instead of local `mock-paid`, include customer fields:

```bash
skillflower checkout-sessions confirm \
  --session-id session-1 \
  --payment-token "$SKILLFLOWER_PAYMENT_TOKEN" \
  --customer-name "Alex Chen" \
  --customer-email "alex@example.com" \
  --customer-phone "+12065550000" \
  --customer-address1 "500 Sender Ave" \
  --customer-address2 "Apt 8" \
  --customer-city "Seattle" \
  --customer-state WA \
  --customer-country US \
  --customer-postal-code 98109 \
  --customer-ip 203.0.113.10
```

Supported commands:

- `skillflower senders create`
- `skillflower receivers create`
- `skillflower receiver-events create`
- `skillflower products recommend`
- `skillflower checkout-sessions create`
- `skillflower checkout-sessions confirm`
- `skillflower orders get`

## Agent Usage Notes

The CLI is a good fit when your Agent wants:

- a stable executable entrypoint instead of hand-written HTTP calls
- JSON output without extra formatting flags
- local validation for required command flags before network dispatch
- non-zero failures with API error text on stderr

## Local Full-Workflow Smoke

After starting Firestore and the Next.js app locally, run the Agent workflow through the CLI:

```bash
export SKILLFLOWER_BASE_URL="http://127.0.0.1:3000"
export SKILLFLOWER_API_KEY="${SKILLFLOWER_API_KEYS%%,*}"
npm run smoke:agent
```

The script executes:

```text
sender -> receiver -> event -> recommend -> checkout -> confirm -> order
```

It prints one JSON document with created IDs, the checkout URL, order status, and each CLI step response. Local placeholder payment mode uses `mock-paid` by default; pass a different token only when testing a non-placeholder payment provider.

For an end-to-end local simulation, start the explicit Florist One mock provider and point the app at it before running the smoke:

```bash
npm run mock:florist-one
FIRESTORE_EMULATOR_HOST=127.0.0.1:8080 FLORIST_ONE_BASE_URL=http://127.0.0.1:31337 npm run dev
npm run smoke:agent
```

Without `FLORIST_ONE_BASE_URL`, checkout confirmation uses the configured real Florist One host and provider failures remain explicit.
