# Skillflower

Skillflower is an agent-first flower gifting API and CLI for OpenClaw agents.
It supports a stable workflow:

```text
sender -> receiver -> event -> recommend -> checkout -> confirm -> order
```

This repository contains the public OpenClaw marketplace information and skill
package. It intentionally does not contain private provider credentials,
deployment `.env` files, or real provider evidence.

## OpenClaw Skill

The skill package is in:

```text
skills/skillflower/SKILL.md
```

It expects:

- `SKILLFLOWER_BASE_URL`
- `SKILLFLOWER_API_KEY`
- `skillflower` CLI from `@skillflower/cli`

## CLI

Install:

```bash
npm install -g @skillflower/cli
```

Configure:

```bash
export SKILLFLOWER_BASE_URL="https://your-skillflower-host"
export SKILLFLOWER_API_KEY="your-public-api-key"
```

Check commands:

```bash
skillflower --help
```

## Docs

- [API](docs/API.md)
- [CLI](docs/CLI.md)
- [OpenClaw marketplace notes](docs/OPENCLAW_MARKETPLACE.md)
- [LLM integration notes](llm.txt)

## Safety

Skillflower does not collect raw card data in chat. Checkout confirmation uses
tokenized payment values. Provider failures must surface explicitly; agents
should not invent products, checkout sessions, order ids, or successful orders.
