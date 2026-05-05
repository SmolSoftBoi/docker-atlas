# Architecture

Docker Atlas is a catalogue-first repository for reusable Docker Compose deployments.

It is designed to make Compose entries easy to find, review, validate, deploy, back up, and evolve.

## Goals

Docker Atlas aims to:

- provide reusable Docker Compose app and stack entries
- keep catalogue entries consistent and reviewable
- document deployment risks before use
- make backup, restore, update, and security expectations visible
- support future exports to tools such as Portainer, Dockge, CasaOS, or a static web catalogue

## Non-goals

Docker Atlas does not aim to be:

- a one-click production installer
- a replacement for upstream app documentation
- a secret store
- a guarantee that every entry is production-ready
- a place for host-specific personal configuration unless clearly isolated in `profiles/`

## Repository model

```text
Docker Atlas
├── apps/        Single application entries
├── stacks/      Multi-service deployment patterns
├── shared/      Reusable Compose fragments and common patterns
├── profiles/    Host or environment-specific overrides
├── templates/   Metadata and README templates
├── standards/   Style, security, and catalogue rules
├── docs/        Supporting documentation
└── .github/     Issue forms, PR template, and validation workflows
```

## Apps

Apps are single deployable services.

Example:

```text
apps/uptime-kuma/
  compose.yaml
  .env.example
  metadata.yaml
  README.md
```

Apps may document optional integrations, but they should not bundle unrelated services by default.

## Stacks

Stacks are grouped deployment patterns where multiple services belong together operationally.

Example:

```text
stacks/media/
  compose.yaml
  .env.example
  metadata.yaml
  README.md
```

Good stack candidates include:

- automation: n8n + Postgres + Redis
- media: Jellyfin or Plex + Sonarr + Radarr + Prowlarr + qBittorrent
- private AI: Open WebUI + optional n8n + optional local model endpoint

## Shared components

`shared/` is for reusable Compose fragments and common deployment patterns.

Examples may include:

- proxy networks
- common environment blocks
- healthcheck patterns
- reusable monitoring sidecars

Shared components should be documented because they may be included by multiple entries.

## Profiles

`profiles/` is for host-specific or environment-specific variants.

Examples:

- Synology-specific paths
- GPU or hardware acceleration variants
- reverse proxy variants
- VPN-bound download profiles

Profiles should keep host-specific assumptions out of the base app or stack entry.

## Templates

`templates/` contains reusable starting points for entries.

Current template types:

- `templates/metadata.yaml`
- `templates/README.template.md`

Templates define the expected shape for future catalogue entries.

## Standards

`standards/` and `docs/` define the operating rules for the catalogue.

Important files:

- `standards/compose-style-guide.md`
- `standards/security-checklist.md`
- `docs/catalogue-standard.md`

These documents are the source of truth for style, security, and review expectations.

## Entry lifecycle

Catalogue entries should move through a simple lifecycle:

```text
issue → draft entry → validation → tested → recommended
```

Suggested status values in `metadata.yaml`:

- `draft` — entry exists but has not been fully reviewed
- `tested` — Compose validates and has been tested locally
- `recommended` — reviewed and suitable for regular use
- `deprecated` — retained for reference only

## Required entry files

Each app or stack entry should include:

```text
compose.yaml
.env.example
metadata.yaml
README.md
```

### `compose.yaml`

The runnable Compose definition.

### `.env.example`

Safe example configuration values and placeholders.

### `metadata.yaml`

Machine-readable catalogue data used for browsing, filtering, review, and future export.

### `README.md`

Human-readable deployment, maintenance, backup, restore, update, and security notes.

## Validation model

Every Compose file should pass:

```bash
docker compose -f path/to/compose.yaml config --quiet
```

The repository validation workflow scans these folders:

```text
apps/
stacks/
shared/
profiles/
```

A Compose file should be valid before a PR is marked ready for review.

## Security model

Docker Atlas treats these as high-risk patterns:

- Docker socket access
- host networking
- privileged mode
- exposed admin web UIs
- default credentials
- persistent sensitive data
- public tunnel or reverse proxy exposure
- hardware or device mounts

Risky patterns are allowed only when they are needed and clearly documented.

## Backup and restore model

Any stateful entry must document:

- which volumes or paths store state
- what should be backed up
- how to stop the stack safely before backup or restore
- restore order for multi-service stacks
- any database-specific notes

Backups should be treated as incomplete until restore steps are documented.

## Naming model

Use lowercase kebab-case for folders:

```text
apps/open-webui
apps/cloudflare-tunnel
stacks/private-ai
stacks/media
```

Use clear top-level Compose project names where appropriate:

```yaml
name: open-webui
```

## Future extension points

Docker Atlas may later add:

- generated catalogue index files
- JSON export
- Portainer template export
- Dockge import helpers
- CasaOS package export
- static web UI
- compatibility matrix by architecture or host type
- automated metadata linting

These should build on the existing catalogue model rather than replacing it.

## Footguns

Watch for these common problems:

- using `latest` tags for important stateful services without documenting the risk
- exposing admin UIs directly to the internet
- hiding required setup in undocumented environment variables
- mixing host-specific paths into base entries
- creating stacks where a single app entry would be clearer
- creating app entries that secretly depend on unrelated bundled services
- documenting backup paths without restore steps
