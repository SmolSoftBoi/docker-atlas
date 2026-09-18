# Uptime Kuma

[Uptime Kuma](https://uptime.kuma.pet/) monitors service availability and provides status pages. This entry uses the full upstream `louislam/uptime-kuma:next` image with local persistent storage and a UI bound to loopback.

## What it includes

- One Uptime Kuma container on an app bridge network.
- The full image, including Chromium for browser monitors and optional embedded MariaDB support. The setup below uses SQLite.
- A dedicated host directory mounted directly at `/app/data`.
- The upstream image's built-in healthcheck.

## Requirements

- Docker Engine and Docker Compose v2 or newer; the image supports AMD64 and ARM64.
- A `.env` file based on `.env.example`.
- A dedicated local data directory with reliable POSIX file locking. Avoid NFS and other storage with unreliable locks: SQLite corruption is possible. See the [upstream installation guide](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install).
- Run the commands below from this entry's directory on the Docker host, with access to the configured data directory. Backup and restore use temporary containers from the deployed image to preserve ownership; the host needs `tar` to check archives.

## Setup

From the repository root:

```bash
cd apps/uptime-kuma
cp .env.example .env
chmod 600 .env
# Review .env before starting, particularly DATA_ROOT and BIND_ADDRESS.
```

Create the container without starting the application, then prepare its resolved data path with owner-only access. This honours `.env` and shell overrides, including relative paths and paths containing spaces. Run this block after reviewing `.env`:

```bash
(
  set -eu
  docker compose config --quiet
  docker compose create
  kuma_container_id="$(docker compose ps -aq uptime-kuma)"
  test -n "$kuma_container_id"
  kuma_data_dir="$(docker inspect --format '{{range .Mounts}}{{if eq .Destination "/app/data"}}{{.Source}}{{end}}{{end}}' "$kuma_container_id")"
  test -n "$kuma_data_dir"
  mkdir -p "$kuma_data_dir"
  chmod 700 "$kuma_data_dir"
  docker compose up -d
  docker compose ps
)
```

The directory must be owned by the account running these host-side maintenance commands. A rootful Docker daemon may create a missing directory as root on Linux. If it belongs to another account, have its owner or an administrator correct ownership and apply mode `0700` before continuing; do not widen access to work around a permission error. For an existing deployment, stop it first and run the preparation block without replacing `.env`. Restricting the directory protects the database and credentials even when files inside it are readable by other users.

Open `http://127.0.0.1:3001`. Select **SQLite** on the database setup screen, then create an administrator account with a strong, unique password. Complete setup before making the service accessible through a proxy. Add a monitor for a service reachable from the container and confirm it reports the expected status.

For a remote Docker host, keep the loopback default and forward the port from your workstation:

```bash
ssh -N -L 3001:127.0.0.1:3001 user@docker-host
```

Then open `http://127.0.0.1:3001` on your workstation. Replace `user@docker-host` and adjust the forwarded remote port if you changed `UPTIME_KUMA_PORT`.

## Configuration

| Variable | Required | Default | Description |
|---|---:|---|---|
| `BIND_ADDRESS` | No | `127.0.0.1` | Host interface for the UI; changing it can expose the service beyond the host. |
| `UPTIME_KUMA_PORT` | No | `3001` | Published host TCP port. |
| `DATA_ROOT` | No | `./data` | Dedicated local directory mapped directly to `/app/data`; no `uptime-kuma` subdirectory is appended. |
| `TZ` | No | `Europe/London` | Container timezone. |

Relative data paths resolve from this entry's directory. Use a directory containing only Uptime Kuma state. `.env`, the default `data/` directory and `backups/` are ignored by Git. Keep custom data/backup paths outside the checkout, or add suitable local ignore rules before deployment; do not commit state or credentials.

The image deliberately follows the moving `next` tag. Pulling later can select a different release, potentially including a major upgrade. Review release notes and migration guidance before updates; see [upstream Docker tags](https://github.com/louislam/uptime-kuma/wiki/Docker-Tags). Updates are operator initiated; this entry does not install an automatic updater.

## Ports

| Host | Container | Purpose |
|---|---:|---|
| `127.0.0.1:3001` by default | `3001/tcp` | Web UI, status pages and WebSocket connections. |

## Volumes

| Host path | Container path | Purpose | Backup required |
|---|---|---|---:|
| `${DATA_ROOT:-./data}` | `/app/data` | Database, settings, monitor history, uploads and credentials. | Yes |

Back up the whole directory, including hidden files and database sidecar files. V2 removed the old JSON backup/restore feature; it is not a substitute for a data backup. See the [migration guide](https://github.com/louislam/uptime-kuma/wiki/Migration-From-v1-To-v2).

## Networks and reverse proxy

The `app` bridge network is managed by this Compose project. Outbound DNS and network access are needed for monitored targets and notification providers. Inside the container, `localhost` refers to Uptime Kuma itself; use reachable target addresses or Docker service names on a shared network.

Proxy integration is optional and not bundled:

- A proxy running directly on the Docker host can forward to `http://127.0.0.1:3001` (or the configured host port).
- A containerised proxy needs a shared Docker network with Uptime Kuma. Add that network through a local Compose override, retain `app`, and use `http://uptime-kuma:3001` as the upstream. Its own `localhost` cannot reach this container. Declare a cross-project network as external in both projects, create it before deployment, and restrict membership: attached containers can reach each other. Use a unique network alias if another service already uses `uptime-kuma`.
- Configure HTTPS, WebSocket forwarding and a dedicated hostname such as `status.example.com`. Uptime Kuma does not support being served under a subdirectory such as `/uptime-kuma`.
- Keep administrator authentication enabled. Public status pages should expose only deliberately selected information; a reverse proxy alone does not add authentication.
- Enable trusted proxy headers only when access to Uptime Kuma is restricted to the trusted proxy.

Follow the [upstream reverse-proxy guide](https://github.com/louislam/uptime-kuma/wiki/Reverse-Proxy) for your proxy. If using an explicitly named override, pass the same `-f` files to all deployment and maintenance commands below.

## Security notes

- The UI binds to loopback by default. Do not publish the administration service directly to the internet; use an HTTPS reverse proxy and retain authentication.
- The container uses the upstream full image's root user and writable filesystem to preserve upstream behaviour, including its bundled dependencies. No privileged mode, host networking or Docker socket mount is configured. Capability restrictions, rootless images and a read-only root filesystem need separate compatibility testing.
- Docker-container monitors require additional Docker API access and are outside this entry's default configuration. Do not add a socket mount casually: it grants extensive control over the Docker host.
- Keep the data directory at mode `0700` and `.env` at mode `0600`, as in **Setup**. They and the backups can contain notification credentials, monitor secrets and private service addresses. Encrypt backups copied off the host.

## Backup

Run this before an update. The helper container reads the deployed container's actual mount, so it also honours a custom `DATA_ROOT`. It uses the currently deployed image and has no network access. The service stays stopped afterwards; run `docker compose start uptime-kuma` if you are only taking a backup.

The backup directory must be outside `DATA_ROOT`. The default `backups/` and `data/` directories are siblings. This example checks the resolved host paths before stopping the service:

```bash
(
  set -eu
  umask 077
  kuma_container_id="$(docker compose ps -aq uptime-kuma)"
  test -n "$kuma_container_id"
  kuma_data_dir="$(docker inspect --format '{{range .Mounts}}{{if eq .Destination "/app/data"}}{{.Source}}{{end}}{{end}}' "$kuma_container_id")"
  kuma_data_dir="$(cd "$kuma_data_dir" && pwd -P)"
  mkdir -p backups
  kuma_backup_root="$(cd backups && pwd -P)"
  case "$kuma_backup_root/" in
    "$kuma_data_dir/"*) echo 'Backups must be outside DATA_ROOT.' >&2; exit 1 ;;
  esac
  kuma_backup_dir="$kuma_backup_root/$(date -u +%Y%m%dT%H%M%SZ)"
  mkdir "$kuma_backup_dir"
  kuma_image_id="$(docker inspect --format '{{.Image}}' "$kuma_container_id")"
  docker image inspect --format '{{range .RepoDigests}}{{println .}}{{end}}' "$kuma_image_id" > "$kuma_backup_dir/image-digests.txt"
  test -s "$kuma_backup_dir/image-digests.txt"
  cp .env "$kuma_backup_dir/deployment.env"
  docker compose stop uptime-kuma
  docker run --rm --network none --volumes-from "$kuma_container_id:ro" \
    --entrypoint tar "$kuma_image_id" -C /app/data -cpf - . > "$kuma_backup_dir/data.tar"
  tar -tf "$kuma_backup_dir/data.tar" > /dev/null
  printf 'Backup saved to %s; Uptime Kuma is stopped.\n' "$kuma_backup_dir"
)
```

Keep the archive, deployment settings and `image-digests.txt` together. Also retain any local Compose overrides alongside the backup. Copy successful backups to protected storage on another device and practise a restore; an archive listing alone does not verify recovery.

## Restore

Choose a trusted backup and review its saved deployment settings. Every restore must use that backup's recorded image digest, including routine recovery and rollback after a failed update. `--pull never` alone cannot guarantee this: the local `next` tag may already point to a different image.

Create `backups/` if needed, then create `backups/restore.compose.yaml` and copy the `louislam/uptime-kuma@sha256:...` reference from the chosen backup's `image-digests.txt` into it:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma@sha256:REPLACE_WITH_RECORDED_DIGEST
```

Validate the complete file set before changing data:

```bash
docker compose -f compose.yaml -f backups/restore.compose.yaml config --quiet
```

Include any existing proxy override before the restore file in every command below. On a replacement host without an existing container, restore the saved configuration and run the **Setup** preparation block with this complete file set, omitting its final `up -d` and `ps` lines. This creates the container and protects the configured data directory without starting the application or initialising a new database.

The helper restores into the existing container's actual data directory. It moves all current data, including hidden files, into the backup directory before extracting, preserving it for recovery. It reapplies mode `0700` after extraction because an older backup may contain broader directory permissions.

Set `kuma_backup_dir` to an absolute path outside `DATA_ROOT`:

```bash
(
  set -eu
  kuma_backup_dir=/absolute/path/to/backups/20260918T120000Z
  kuma_backup_dir="$(cd "$kuma_backup_dir" && pwd -P)"
  test -f "$kuma_backup_dir/data.tar"
  tar -tf "$kuma_backup_dir/data.tar" > /dev/null
  kuma_container_id="$(docker compose -f compose.yaml -f backups/restore.compose.yaml ps -aq uptime-kuma)"
  test -n "$kuma_container_id"
  kuma_data_dir="$(docker inspect --format '{{range .Mounts}}{{if eq .Destination "/app/data"}}{{.Source}}{{end}}{{end}}' "$kuma_container_id")"
  kuma_data_dir="$(cd "$kuma_data_dir" && pwd -P)"
  case "$kuma_backup_dir/" in
    "$kuma_data_dir/"*) echo 'Backups must be outside DATA_ROOT.' >&2; exit 1 ;;
  esac
  kuma_previous_name="previous-data-$(date -u +%Y%m%dT%H%M%SZ)"
  kuma_image_id="$(docker inspect --format '{{.Image}}' "$kuma_container_id")"
  docker compose -f compose.yaml -f backups/restore.compose.yaml stop uptime-kuma
  docker run --rm --network none --volumes-from "$kuma_container_id" \
    --mount "type=bind,src=$kuma_backup_dir,dst=/backup" \
    --entrypoint sh "$kuma_image_id" -ec '
      mkdir "/backup/$1"
      find /app/data -mindepth 1 -maxdepth 1 -exec mv -t "/backup/$1" -- {} +
      tar --numeric-owner -xpf /backup/data.tar -C /app/data
      chmod 700 /app/data
    ' sh "$kuma_previous_name"
)
```

Keep `.env` pointing to the restored directory. Recreate using the same digest override, pulling the recorded image if it is not cached:

```bash
docker compose -f compose.yaml -f backups/restore.compose.yaml up -d --force-recreate
docker compose -f compose.yaml -f backups/restore.compose.yaml ps
docker compose -f compose.yaml -f backups/restore.compose.yaml logs --tail=100 uptime-kuma
```

Confirm the container becomes healthy, the administrator account works, and the expected monitors and history are present.

Continue using the complete file set for subsequent maintenance until intentionally returning to `next`; a plain `docker compose up -d` would select the moving tag again.

## Update

1. Review the [release notes](https://github.com/louislam/uptime-kuma/releases) and applicable migration guidance, including any major-version changes since deployment.
2. Complete **Backup** above. This records the deployed image digest before `next` changes and leaves the service stopped. Do not continue if backup fails.
3. Pull and recreate:

   ```bash
   docker compose pull
   docker compose up -d
   docker compose ps
   docker compose logs --tail=100 uptime-kuma
   ```

4. Allow database migrations to finish without interruption. Confirm health, administrator login and monitor results. Retain the pre-update backup until these checks pass.

### Rollback

Follow **Restore** using the matching pre-update backup and its recorded image digest. The same `backups/restore.compose.yaml` override handles rollback and routine recovery. Restore the image and data together: downgrading the image alone is insufficient after a database migration. Verify login, health and monitors before intentionally returning to `next` through a later update.

## Validate

Configuration validation does not start containers:

```bash
docker compose config --quiet
docker compose --env-file .env.example config --quiet
docker compose config --profiles
```

This entry has no profiles. Check that the rendered configuration uses the intended bind address and mounts `DATA_ROOT` directly at `/app/data`.

Before promoting this entry to `tested`, use an isolated Compose project, temporary data directory and unused loopback port to verify:

1. SQLite setup, administrator creation, authenticated access and container health.
2. An HTTP monitor against a controlled endpoint reachable from the app network.
3. Account and monitor persistence after container recreation.
4. A stopped backup restored into separate temporary storage, with login and monitoring verified again.

Record the tested image digest and architecture, because `next` can change independently of these files.

### Validation evidence

Tested on 2026-09-18 with Docker Desktop on ARM64 and Docker Compose 5.5.1. The `next` image reported Uptime Kuma 2.5.5 and resolved to:

```text
louislam/uptime-kuma@sha256:c74379ac4509ce2d2c2633f509e67003ee2e45b6e995c5e43fc101f45a0e1fbe
```

- Default, example and overridden configuration passed, including a custom data path containing spaces. All repository Compose files and their profiles validated.
- Setup applied mode `0700` to the default and custom host data directories before startup, retained the host user's ownership, and restricted `.env` to mode `0600`.
- SQLite setup, administrator creation, rejection of unauthenticated access and invalid credentials, HTTP monitoring, and the upstream healthcheck passed.
- Account and monitor state survived container recreation. The documented stopped backup and restore recovered state into separate storage, preserved hidden files and numeric ownership, and moved pre-existing data aside.
- The restore override selected the recorded image digest despite a simulated change to the cached moving tag. The restored deployment passed login, monitoring and health checks, and an archive with a `0755` data directory was restored with mode `0700`.

Runtime checks covered SQLite and HTTP monitoring on ARM64. Host ownership and permission checks used macOS; native Linux host permissions, AMD64 and external proxy integration were not exercised. This evidence applies to the recorded image; future `next` images require fresh validation.
