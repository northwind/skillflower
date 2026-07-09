# OpenClaw Marketplace Notes

Maintainers should publish the repository root. The installable skill file is:

```text
SKILL.md
```

## Listing Copy

```text
Order florist-backed flowers with an OpenClaw agent. Send Flowers collects
recipient details, recommends a bouquet, creates checkout, returns a browser
payment link, and checks order status through the hosted Send Flowers service.
```

## Dry Run

```bash
clawhub skill publish . \
  --slug send-flowers \
  --name "Send Flowers" \
  --version 0.1.0 \
  --dry-run
```

Publish only after the dry run shows the expected owner, slug, files, version,
and changelog.

## Public Package Boundary

Keep this repository limited to:

- `SKILL.md`
- user install instructions
- ordering contract
- marketplace notes
- `llm.txt`

Do not publish private application source, `.env` files, provider credentials,
provider evidence, deployment details, admin workflows, or internal test output.

## Runtime Configuration

The public skill includes the default hosted base URL:

```text
https://skillflower.goosepod.org
```

Users only need a Send Flowers API key. The skill may cache that key at
`~/.send-flowers/api-key`, but it must not generate random local keys unless the
hosted service adds an explicit key-registration endpoint that accepts them.
