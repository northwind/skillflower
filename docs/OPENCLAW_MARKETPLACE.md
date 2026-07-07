# OpenClaw Marketplace Notes

This repository is the public Skillflower skill package. Publish the repository
root, because `SKILL.md` is at the root.

Dry run:

```bash
clawhub skill publish . \
  --slug skillflower \
  --name "Skillflower" \
  --version 0.1.0 \
  --dry-run
```

Publish only after the dry run shows the expected owner, slug, files, version,
and changelog.

The marketplace listing should describe the user-facing capability:

```text
Order florist-backed flowers with an OpenClaw agent. The skill collects
recipient details, recommends a bouquet, creates checkout, and returns order
status through the hosted Skillflower service.
```

Do not publish private application source, provider credentials, `.env` files,
test evidence, admin workflows, or internal deployment docs from the private
Skillflower app repository.
