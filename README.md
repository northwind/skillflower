# Skillflower OpenClaw Skill

Skillflower lets an OpenClaw agent help a user order florist-backed flowers.
This repository is the public skill distribution surface for users and agents.
It is not the private application source repository.

The installed skill supports the ordering path:

```text
collect recipient details -> recommend bouquet -> create checkout -> finish payment -> track order
```

## What Users Can Do

Ask an agent things like:

```text
Send birthday flowers to my mom in Seattle next Tuesday. Keep it under $120 and make it warm but not flashy.
```

The skill guides the agent to:

- collect sender, recipient, address, occasion, delivery date, tone, and budget
- recommend a florist-backed bouquet
- create a checkout session
- return a browser checkout link or confirm a tokenized payment
- look up the resulting order

## Install

Install the skill file into your OpenClaw workspace:

```bash
mkdir -p ~/.openclaw/workspace/skills/skillflower
curl -fsSL https://raw.githubusercontent.com/northwind/skillflower/main/SKILL.md \
  -o ~/.openclaw/workspace/skills/skillflower/SKILL.md
```

Restart OpenClaw or start a new chat so the skill list refreshes.

## Configure

Skillflower needs a hosted service URL and API key:

```bash
export SKILLFLOWER_BASE_URL="https://your-skillflower-host"
export SKILLFLOWER_API_KEY="your-skillflower-api-key"
```

Use the service URL and key issued by the Skillflower operator. Without both,
the skill must stop before creating orders.

## Payment Safety

Do not enter raw card number, CVV, or raw bank data in chat. Skillflower uses
browser checkout handoff or tokenized payment values only.

## For Agents

Read [SKILL.md](SKILL.md) for the executable ordering contract and
[docs/ORDERING_CONTRACT.md](docs/ORDERING_CONTRACT.md) for the public API shape
the skill relies on.

## Scope

This public repo intentionally contains only the installable skill and
external ordering docs. Internal implementation, provider credentials,
deployment config, test evidence, and admin workflows live outside this repo.
