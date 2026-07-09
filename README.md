# Send Flowers Universal Agent Skill

Send Flowers is a portable skill for Coding Agents and agentic IDEs. It lets
an agent help a user send florist-backed flowers by collecting delivery details,
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
mkdir -p ~/.openclaw/workspace/skills/send-flowers
curl -fsSL https://raw.githubusercontent.com/northwind/send-flowers/main/SKILL.md \
  -o ~/.openclaw/workspace/skills/send-flowers/SKILL.md
```

Restart OpenClaw or start a new chat so the skill list refreshes.

### Claude Code, Codex, Antigravity, And Similar Agents

Add this repository's [SKILL.md](SKILL.md) to the agent runtime's local skills
folder under a `send-flowers` directory. If the runtime supports importing a
skill from a GitHub URL, use:

```text
https://raw.githubusercontent.com/northwind/send-flowers/main/SKILL.md
```

If the runtime only supports project-local instructions, copy `SKILL.md` into a
project skill/instructions folder and name it `send-flowers`.

## Configure

The hosted service URL is built into the skill:

```bash
https://skillflower.goosepod.org
```

Override it only when the operator gives you a different host:

```bash
export SEND_FLOWERS_BASE_URL="https://another-send-flowers-host"
```

Ask the Send Flowers operator for an API key, then either export it:

```bash
export SEND_FLOWERS_API_KEY="your-send-flowers-api-key"
```

or cache it locally:

```bash
mkdir -p ~/.send-flowers
printf '%s\n' "your-send-flowers-api-key" > ~/.send-flowers/api-key
chmod 600 ~/.send-flowers/api-key
```

The skill can be installed before the API key is set. It must not create
checkout sessions or orders until a valid API key is available.

The API key cannot be generated purely inside `SKILL.md`: the hosted service
must issue or accept the key. A random local key will be rejected.

## How Agents Should Use It

1. Read [SKILL.md](SKILL.md).
2. Use the built-in base URL unless `SEND_FLOWERS_BASE_URL` overrides it.
3. Load `SEND_FLOWERS_API_KEY` from env or `~/.send-flowers/api-key`.
4. Collect only the order data needed from the user.
5. Call the hosted Send Flowers API as described in the skill.
6. Return the checkout link for browser payment.
7. Check order status only after payment is complete.

## Limits

- US and Canada delivery addresses only.
- Browser checkout is the default public payment path.
- Tokenized payment confirmation is only for approved payment integrations.
- The skill does not collect raw card number, CVV, or bank data in chat.
- The skill does not expose provider credentials, admin workflows, deployment
  config, or private application source.
- If the Send Flowers API returns an error, the agent must show the exact error
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
