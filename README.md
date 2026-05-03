# Docker Atlas

**A curated map of reusable Docker Compose stacks, standards, and deployment patterns.**

Docker Atlas is a practical catalogue for reusable `compose.yaml` files, homelab stacks, deployment recipes, and lightweight standards.

## Purpose

Docker Atlas helps you:

- reuse Docker Compose app definitions consistently;
- keep deployment notes, metadata, and environment examples together;
- validate Compose files before use;
- build a future-friendly base for Portainer, Dockge, CasaOS, or a browsable catalogue UI.

## Structure

```text
apps/        Single-app Compose templates
stacks/      Multi-service stack templates
shared/      Reusable Compose fragments and common networks
profiles/    Host or environment-specific overrides
templates/   Reusable metadata and documentation templates
standards/   Style, security, and catalogue rules
docs/        Supporting documentation
scripts/     Helper scripts
```

## App entry standard

Each app should use this structure:

```text
apps/example-app/
  compose.yaml
  .env.example
  metadata.yaml
  README.md
```

## Quick validation

```bash
docker compose -f apps/example-app/compose.yaml config --quiet
```

## Core conventions

- Use `compose.yaml` as the canonical Compose file name.
- Do not use the obsolete top-level `version` property.
- Keep real secrets out of Git; commit `.env.example` only.
- Prefer pinned image versions for important services.
- Use `${DATA_ROOT}` for host paths instead of hardcoded machine paths.
- Document backup, restore, ports, volumes, and exposure risks.

## Status

Early project scaffold. First catalogue entries coming next.
