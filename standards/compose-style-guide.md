# Compose Style Guide

Docker Atlas uses the Compose Specification as the baseline for reusable Compose files.

## File naming

Use `compose.yaml` as the canonical file name.

Avoid legacy names unless compatibility requires them:

- `docker-compose.yml`
- `docker-compose.yaml`

## Top-level version

Do not include the obsolete top-level `version` property.

Use:

```yaml
name: example-app

services:
  app:
    image: example/example-app:1.0.0
```

Avoid:

```yaml
version: "3.8"
```

## Project name

Set a clear top-level `name` value using lowercase letters, numbers, dashes, or underscores.

```yaml
name: n8n
```

## Environment variables

Use variable interpolation with safe defaults where appropriate.

```yaml
environment:
  TZ: ${TZ:-Europe/London}
```

Use `.env.example` for documented defaults. Do not commit real `.env` files.

## Host paths

Use `${DATA_ROOT}` for host-mounted app data.

```yaml
volumes:
  - ${DATA_ROOT:-./data}/app:/config
```

Avoid hardcoding machine-specific paths unless documented as a host profile.

## Images

Prefer pinned versions for important services.

```yaml
image: postgres:16
```

Use `latest` only for experiments or where explicitly justified in the app README.

## Restart policy

For persistent homelab services, prefer:

```yaml
restart: unless-stopped
```

## Networks

Prefer explicit app networks.

```yaml
networks:
  app_net:
```

Use an external reverse-proxy network only when documented.

## Ports

Expose only required ports. Prefer environment variables for host ports.

```yaml
ports:
  - "${APP_PORT:-8080}:8080"
```

## Metadata

Every catalogue app should include:

- `metadata.yaml`
- `.env.example`
- `README.md`
- `compose.yaml`
