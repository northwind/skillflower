# Skillflower OpenClaw Skill

Skillflower lets an OpenClaw agent help a user send florist-backed flowers.
The skill collects delivery details, recommends a bouquet, creates checkout,
and returns either a browser payment link or order status.

## Install

Manual install from GitHub:

```bash
mkdir -p ~/.openclaw/workspace/skills/skillflower
curl -fsSL https://raw.githubusercontent.com/northwind/skillflower/main/SKILL.md \
  -o ~/.openclaw/workspace/skills/skillflower/SKILL.md
```

Then restart OpenClaw or start a new chat so the skill list refreshes.

After this package is published on ClawHub, users can install it from the
registry instead.

## Configure

Ask the Skillflower operator for a service URL and API key, then set:

```bash
export SKILLFLOWER_BASE_URL="https://your-skillflower-host"
export SKILLFLOWER_API_KEY="your-skillflower-api-key"
```

The skill stays visible even before these values are configured, but it will
not create checkout sessions or orders until both are present.

## Use

Ask your OpenClaw agent:

```text
Send birthday flowers to my mom in Seattle next Tuesday. Keep it under $120 and make it warm but not flashy.
```

The agent will:

- collect sender and recipient details
- collect delivery address, occasion, date, tone, and budget
- recommend a bouquet
- create checkout
- return a browser checkout link
- check the order after payment completes

## Payment Safety

Do not enter raw card number, CVV, or bank data in chat. The public flow uses a
browser checkout link. Tokenized payment confirmation is only for approved
payment integrations that already produced a token.

## Files

- [SKILL.md](SKILL.md): installable OpenClaw skill.
- [references/ORDERING_CONTRACT.md](references/ORDERING_CONTRACT.md): API shape used by the skill.
- [references/OPENCLAW_MARKETPLACE.md](references/OPENCLAW_MARKETPLACE.md): maintainer publishing notes.
- [llm.txt](llm.txt): short machine-readable summary.

This repository is intentionally small. Private application source, provider
credentials, deployment settings, and test evidence are not part of the public
skill package.
