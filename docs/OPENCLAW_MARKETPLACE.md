# OpenClaw Marketplace Notes

Skillflower's OpenClaw skill package lives at:

```text
skills/skillflower/SKILL.md
```

The package expects:

- `SKILLFLOWER_BASE_URL`
- `SKILLFLOWER_API_KEY`
- the `skillflower` CLI from `@skillflower/cli`

Local checks:

```bash
npm test
npm run build
npm run test:e2e
npm pack --workspace @skillflower/cli --dry-run
npm exec --workspace @skillflower/cli -- skillflower --help
```

OpenClaw/ClawHub checks, when those CLIs are installed:

```bash
openclaw skills list
clawhub skill publish ./skills/skillflower --slug skillflower --name "Skillflower" --version 0.1.0 --dry-run
```

Publish for real only after the dry run shows the intended owner, slug, files,
version, and changelog.

Do not publish the private app repository as-is. The full app repo can contain
provider credentials, `.env`, and real provider evidence. Use the public
`northwind/skillflower` information repository for marketplace-facing docs and
the skill package.
