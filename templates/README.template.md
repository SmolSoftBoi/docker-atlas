# Example App

Short description of what this app does and why someone would deploy it.

## What it includes

- App container
- Persistent volume
- Optional reverse proxy support

## Requirements

- Docker Engine
- Docker Compose v2
- A configured `.env` file based on `.env.example`

## Setup

```bash
cp .env.example .env
docker compose up -d
```

## Configuration

| Variable | Required | Default | Description |
|---|---:|---|---|
| `TZ` | No | `Europe/London` | Container timezone |
| `DATA_ROOT` | No | `./data` | Host data root |

## Ports

| Host | Container | Purpose |
|---:|---:|---|
| `8080` | `80` | Web UI |

## Volumes

| Volume / Path | Purpose | Backup required |
|---|---|---:|
| `${DATA_ROOT}/example-app` | Persistent app data | Yes |

## Security notes

- Change default credentials before use.
- Do not expose directly to the internet unless upstream supports it safely.
- Prefer a reverse proxy with HTTPS and authentication for public access.

## Backup

Back up the configured data path before upgrading.

## Restore

1. Stop the stack.
2. Restore the data path or volume.
3. Start the stack.
4. Confirm the application starts cleanly.

## Update

```bash
docker compose pull
docker compose up -d
```

## Validate

```bash
docker compose config --quiet
```
