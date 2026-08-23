# Cloudflare Tunnel

Cloudflare Tunnel runs outbound-only `cloudflared` connectors for remotely managed tunnels without opening inbound ports on the Docker host.

This entry provides separate bridge and host-network connectors. Each connector uses its own Cloudflare tunnel token and Docker hostname.

## What it includes

- A `bridge` profile for an isolated connector on the shared external `cloudflare-tunnel` Docker network.
- A `host` profile for an opt-in connector sharing the Docker host network namespace.
- Direct `TUNNEL_TOKEN` environment injection inside each container, with no token mounts or `--token-file` usage.
- A loopback metrics endpoint used by the container health check.

## Requirements

- Docker Engine and Docker Compose v2 with [profile support](https://docs.docker.com/compose/how-tos/profiles/).
- A Cloudflare account and one remotely managed tunnel for each active profile.
- Outbound DNS plus TCP or UDP connectivity on port `7844` to Cloudflare Tunnel endpoints. Current `cloudflared` releases perform [connectivity pre-checks](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/troubleshoot-tunnels/connectivity-prechecks/) at startup.
- For `host` mode, Docker Engine on Linux or Docker Desktop 4.34 or later with host networking enabled. Review the [Docker host-network limitations](https://docs.docker.com/engine/network/drivers/host/) before enabling it.

## Setup

1. In the Cloudflare dashboard, create the remotely managed tunnel or tunnels and add a published application route for each public hostname.
2. Copy the example environment file and restrict its permissions:

   ```bash
   cp .env.example .env
   chmod 600 .env
   ```

3. Set the token for each selected profile in `.env`. Do not paste tokens into commands, Compose files, logs, or documentation.
4. Select the required profile or profiles with `COMPOSE_PROFILES`.
5. If `bridge` is selected, create its external Docker network once:

   ```bash
   docker network inspect cloudflare-tunnel >/dev/null 2>&1 ||
     docker network create cloudflare-tunnel
   ```

6. Start the connector or connectors:

   ```bash
   docker compose up -d
   docker compose ps
   ```

A missing or invalid token prevents the corresponding active connector from starting successfully or becoming healthy. Inactive-profile tokens may remain blank.

### DNS routing

A published application requires both a hostname-to-origin route and a matching DNS record:

- With a full Cloudflare DNS setup, adding the route in the dashboard automatically creates a DNS record that points the public hostname to `<UUID>.cfargotunnel.com`.
- Routes created through the API need a separate proxied `CNAME` record for each public hostname, also targeting `<UUID>.cfargotunnel.com`. The locally managed `cloudflared tunnel route dns` command can create this record, but it requires the account certificate and is outside this token-only deployment.
- With a partial DNS setup, create a `CNAME` record, or an `ALIAS` at the zone apex, through the current authoritative DNS provider for each hostname. Follow Cloudflare's [partial DNS guidance](https://developers.cloudflare.com/cloudflare-one/faq/cloudflare-tunnels-faq/#how-can-tunnel-be-used-with-partial-dns-cname-setup) for the required `<public-hostname>.cdn.cloudflare.net` target.

DNS and connector state are independent. A healthy connector does not prove that the public hostname resolves, and a DNS record remains after a connector stops. After deployment, check each record in the authoritative DNS provider and request each published hostname. See Cloudflare's [Tunnel routing documentation](https://developers.cloudflare.com/tunnel/routing/) for the current routing and DNS behaviour.

## Profiles

| Profile | Default | Network behaviour | Use when |
|---|---:|---|---|
| `bridge` | Yes | Shared external bridge network named `cloudflare-tunnel` | Origins are containers on the shared network or are reachable by LAN address |
| `host` | No | Shares the Docker host network namespace | The connector must reach host-local or LAN services through the host network stack |

Set one of the following values in `.env`:

```dotenv
COMPOSE_PROFILES=bridge
COMPOSE_PROFILES=host
COMPOSE_PROFILES=bridge,host
```

When both profiles are selected:

- set both token variables;
- use tokens belonging to two different remotely managed tunnels;
- keep the connector hostnames distinct; and
- configure each tunnel's routes for the origins reachable through its networking mode.

Compose cannot compare secret values, so the operator must ensure the two tokens are different. These connectors are independent and do not form replicas of one tunnel.

You can use repeated CLI flags instead of `COMPOSE_PROFILES`. Explicit CLI profile flags take precedence over the environment selection, so pass every profile that should be active:

```bash
docker compose --profile bridge up -d
docker compose --profile host up -d
docker compose --profile bridge --profile host up -d
```

Before changing profile selections, remove containers from the previous selection:

```bash
docker compose --profile "*" down
```

The external `cloudflare-tunnel` network is not removed by `docker compose down`, so other projects can remain attached across connector profile changes.

### Bridge origins

The `bridge` connector can address another container by service name when that service joins the external `cloudflare-tunnel` network:

```yaml
services:
  example:
    image: example/example:1.0.0
    networks:
      - cloudflare-tunnel

networks:
  cloudflare-tunnel:
    external: true
    name: cloudflare-tunnel
```

Configure its Cloudflare origin URL with the service name and container port, for example `http://example:8080`. Routable LAN origins can use an address such as `http://192.168.1.20:8080`. In bridge mode, `localhost` refers to the `cloudflared` container, not the Docker host.

### Host origins

The `host` connector can reach Linux host-local services through addresses such as `http://127.0.0.1:8080` and can use the host's LAN routing. It does not receive Docker bridge service discovery, and it materially reduces network isolation.

Docker Desktop requires host networking to be enabled explicitly, supports it only for Linux containers, operates at layer 4, and cannot combine it with Enhanced Container Isolation.

## Configuration

| Variable | Required | Default | Description |
|---|---:|---|---|
| `COMPOSE_PROFILES` | No | `bridge` | Activates `bridge`, `host`, or comma-separated `bridge,host` |
| `TUNNEL_TOKEN_BRIDGE` | For `bridge` | None | Token mapped only to the bridge container's `TUNNEL_TOKEN` |
| `TUNNEL_TOKEN_HOST` | For `host` | None | Token mapped only to the host container's `TUNNEL_TOKEN` |
| `TUNNEL_HOSTNAME_BRIDGE` | No | `cloudflare-tunnel-bridge` | Docker hostname for the bridge connector |
| `TUNNEL_HOSTNAME_HOST` | No | `cloudflare-tunnel-host` | Docker hostname for the host connector |
| `METRICS_ADDRESS` | No | `127.0.0.1:20241` | Metrics listener and readiness-check address used by both connectors |

Use lowercase DNS-compatible connector hostnames containing letters, digits, and hyphens. A connector hostname is an operational Docker identifier; it does not select or authenticate a Cloudflare tunnel. The matching token is authoritative for tunnel identity, as documented in Cloudflare's [run parameters](https://developers.cloudflare.com/tunnel/advanced/run-parameters/).

## Ports

This entry publishes no inbound ports. `cloudflared` establishes outbound connections to Cloudflare.

The metrics listener defaults to loopback:

- In bridge mode, `127.0.0.1:20241` exists only inside the connector's network namespace.
- In host mode, it binds to the Docker host's loopback interface.
- When both profiles run, the shared port does not conflict because the bridge connector has its own network namespace.

Do not set `METRICS_ADDRESS` to `0.0.0.0:20241` without access controls. In host mode, that exposes metrics through every suitable host interface.

## Volumes

The remotely managed deployment uses no volumes. Tunnel routes and ingress configuration remain in Cloudflare, and tokens are injected from `.env`.

## Logs and health

View connector state and logs with:

```bash
docker compose ps
docker compose logs -f cloudflared-bridge
docker compose logs -f cloudflared-host
```

The health check runs `cloudflared tunnel --metrics <address> ready`. It reports healthy only after the connector establishes an active connection to Cloudflare. The Cloudflare dashboard should also show the corresponding connector as healthy.

## Security notes

- Anyone with a tunnel token can run that tunnel. Treat both token variables as secrets and rotate either token immediately if exposed.
- Environment-injected secrets are visible to users with sufficient Docker API or `docker inspect` access. Limit Docker access and protect `.env` with restrictive permissions.
- `.env`, `local/`, and `secrets/` are ignored by this entry. Confirm `git status` before every commit.
- A public hostname routed through Cloudflare Tunnel is still public exposure unless Cloudflare Access or another policy restricts it. Keep origin authentication and application hardening enabled.
- Do not disable origin TLS verification merely to bypass certificate errors. Prefer a valid certificate or a trusted private CA.
- Host mode removes network-namespace isolation. Use bridge mode unless the origin genuinely requires the host network stack.
- The containers run read-only with all Linux capabilities dropped and `no-new-privileges` enabled. The upstream image supplies its non-root runtime user.

## Token rotation

Rotate one connector at a time:

1. Rotate or retrieve the selected tunnel token in Cloudflare.
2. Update only its matching variable in `.env`.
3. Recreate that connector:

   ```bash
   docker compose --profile bridge up -d --force-recreate cloudflared-bridge
   docker compose --profile host up -d --force-recreate cloudflared-host
   ```

4. Confirm container health and Cloudflare dashboard status before rotating the other token.

## Backup

The connectors are stateless. Cloudflare stores remotely managed tunnel routes and settings.

Back up `.env` only through an encrypted secret-management or backup system. Do not add it to the repository or an unencrypted archive.

## Restore

1. Recreate or retrieve each remotely managed tunnel token from Cloudflare.
2. Restore `.env` from protected secret storage or rebuild it from `.env.example`.
3. Select the required profiles.
4. Run `docker compose up -d` and confirm the selected container health and dashboard status.

## Update

This entry intentionally follows `cloudflare/cloudflared:latest`, matching Cloudflare's container deployment guidance. A rolling tag can change without a repository diff, so review upstream [releases](https://github.com/cloudflare/cloudflared/releases) before updating.

Update the selected profiles with:

```bash
docker compose pull
docker compose up -d
docker compose ps
```

For rollback, temporarily replace `latest` in `compose.yaml` with the last known-good release tag or digest, pull it, and recreate the selected connectors. Restore `latest` only after the upstream issue is understood.

## Locally managed alternative

Cloudflare recommends remotely managed tunnels for most deployments, and this entry does not ship a runnable local configuration variant.

For an advanced locally managed adaptation, place `config.yml` and the tunnel UUID credential JSON under the ignored `local/` directory and mount only those files read-only. Remove the token environment mapping and run the named tunnel. Never mount the account-wide `cert.pem` into the runtime container. Follow Cloudflare's [locally managed tunnel guidance](https://developers.cloudflare.com/tunnel/advanced/local-management/).

## Validate

Static validation does not contact Cloudflare. Use dummy values rather than real tokens:

```bash
COMPOSE_PROFILES= TUNNEL_TOKEN_BRIDGE=dummy-bridge \
  docker compose --profile bridge config --quiet

COMPOSE_PROFILES= TUNNEL_TOKEN_HOST=dummy-host \
  docker compose --profile host config --quiet

COMPOSE_PROFILES= TUNNEL_TOKEN_BRIDGE=dummy-bridge TUNNEL_TOKEN_HOST=dummy-host \
  docker compose --profile bridge --profile host config --quiet

docker compose config --profiles
```
