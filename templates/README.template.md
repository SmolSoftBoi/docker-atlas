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

## Profiles

Remove this section when the entry does not use Compose profiles. Otherwise, document every profile, the safe default, and whether profiles can be combined.

| Profile | Default | Purpose | Can combine with |
|---|---:|---|---|
| `default-profile` | Yes | Standard deployment | `optional-profile` |
| `optional-profile` | No | Opt-in deployment behaviour | `default-profile` |

Select one or multiple profiles in `.env`:

```dotenv
COMPOSE_PROFILES=default-profile
COMPOSE_PROFILES=default-profile,optional-profile
```

Repeated `--profile` flags are also supported. Explain that explicit CLI profile flags take precedence over `COMPOSE_PROFILES` and must name the complete intended profile set.

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
docker compose config --profiles
docker compose --profile default-profile config --quiet
docker compose --profile "*" config --quiet
```
