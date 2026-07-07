# Skillflower Universal Agent Skill

Skillflower is a portable skill for Coding Agents and agentic IDEs. It lets an
agent help a user send florist-backed flowers by collecting delivery details,
recommending a bouquet, creating checkout, and returning a browser payment link
or order status.

It is designed to work with skill-aware agents such as OpenClaw, Claude Code,
Codex, Antigravity, and similar Coding Agent runtimes that can read a local
`SKILL.md` file and run shell commands.

## What It Does

Ask your agent:

```text
Send birthday flowers to my mom in Seattle next Tuesday. Keep it under $120 and make it warm but not flashy.
```

The skill guides the agent to:

- collect sender, recipient, address, occasion, date, tone, and budget
- recommend a florist-backed bouquet
- create a checkout session
- return a browser checkout link
- check order status after payment completes

## Install

### OpenClaw

```bash
mkdir -p ~/.openclaw/workspace/skills/skillflower
curl -fsSL https://raw.githubusercontent.com/northwind/skillflower/main/SKILL.md \
  -o ~/.openclaw/workspace/skills/skillflower/SKILL.md
```

Restart OpenClaw or start a new chat so the skill list refreshes.

### Claude Code, Codex, Antigravity, And Similar Agents

Add this repository's [SKILL.md](SKILL.md) to the agent runtime's local skills
folder under a `skillflower` directory. If the runtime supports importing a
skill from a GitHub URL, use:

```text
https://raw.githubusercontent.com/northwind/skillflower/main/SKILL.md
```

If the runtime only supports project-local instructions, copy `SKILL.md` into a
project skill/instructions folder and name it `skillflower`.

## Configure

Ask the Skillflower operator for a hosted service URL and API key:

```bash
export SKILLFLOWER_BASE_URL="https://your-skillflower-host"
export SKILLFLOWER_API_KEY="your-skillflower-api-key"
```

The skill can be installed before these values are set. It must not create
checkout sessions or orders until both values are present.

## How Agents Should Use It

1. Read [SKILL.md](SKILL.md).
2. Confirm `curl`, `SKILLFLOWER_BASE_URL`, and `SKILLFLOWER_API_KEY` are available.
3. Collect only the order data needed from the user.
4. Call the hosted Skillflower API as described in the skill.
5. Return the checkout link for browser payment.
6. Check order status only after payment is complete.

## Limits

- US and Canada delivery addresses only.
- Browser checkout is the default public payment path.
- Tokenized payment confirmation is only for approved payment integrations.
- The skill does not collect raw card number, CVV, or bank data in chat.
- The skill does not expose provider credentials, admin workflows, deployment
  config, or private application source.
- If the Skillflower API returns an error, the agent must show the exact error
  and stop. It must not invent products, checkout sessions, provider order ids,
  payment success, or order status.

## Files

- [SKILL.md](SKILL.md): installable universal agent skill.
- [references/ORDERING_CONTRACT.md](references/ORDERING_CONTRACT.md): API shape used by the skill.
- [references/OPENCLAW_MARKETPLACE.md](references/OPENCLAW_MARKETPLACE.md): maintainer publishing notes.
- [llm.txt](llm.txt): short machine-readable summary.

This repository is intentionally small and public. Private application source,
provider credentials, deployment settings, and test evidence are not part of
the skill package.
