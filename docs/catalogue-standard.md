# Catalogue Standard

Docker Atlas entries should be easy to browse, validate, deploy, back up, and review.

## Entry types

### App

A single application or service.

```text
apps/app-name/
  compose.yaml
  .env.example
  metadata.yaml
  README.md
```

### Stack

A grouped deployment containing multiple related services.

```text
stacks/stack-name/
  compose.yaml
  .env.example
  metadata.yaml
  README.md
```

### Shared component

A reusable network, reverse proxy pattern, or common Compose fragment.

```text
shared/component-name/
  compose.yaml
  README.md
```

## Required files

### `compose.yaml`

The runnable Compose definition.

### `.env.example`

Documented environment variables with safe defaults or placeholders.

### `metadata.yaml`

Machine-readable catalogue data for browsing, filtering, review, and future export.

### `README.md`

Human-readable deployment and maintenance notes.

## Status levels

| Status | Meaning |
|---|---|
| `draft` | Entry exists but has not been fully reviewed. |
| `tested` | Compose validates and has been tested locally. |
| `recommended` | Reviewed, documented, and suitable for regular use. |
| `deprecated` | Retained for reference; avoid new deployments. |

## Review expectations

Before moving from `draft` to `tested`:

- `docker compose config --quiet` passes
- required variables are documented
- ports and volumes are documented
- security risks are documented
- backup/restore notes exist for stateful apps

Before moving from `tested` to `recommended`:

- app has been deployed successfully
- README has enough context for another person to use it
- risky permissions are justified
- image source is trusted
- update path is documented

## Naming

Use lowercase kebab-case folder names:

```text
apps/home-assistant
apps/vaultwarden
stacks/media-stack
```

## Categories

Recommended categories:

- automation
- backup
- dashboard
- database
- development
- dns
- document-management
- finance
- home-automation
- identity
- media
- monitoring
- networking
- photo-management
- productivity
- proxy
- security
- storage
