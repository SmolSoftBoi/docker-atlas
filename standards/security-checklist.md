# Security Checklist

Use this checklist before adding or promoting a Docker Atlas catalogue entry.

## Secrets

- [ ] No real `.env` files are committed.
- [ ] No API keys, passwords, tokens, or private URLs are committed.
- [ ] `.env.example` documents every required variable.
- [ ] Default credentials are changed or clearly flagged.

## Images

- [ ] Images come from trusted upstreams.
- [ ] Critical services use pinned tags where practical.
- [ ] The README links to upstream documentation.
- [ ] Deprecated or unmaintained images are avoided or flagged.

## Privileges

- [ ] `privileged: true` is avoided unless strictly required.
- [ ] `network_mode: host` is avoided unless strictly required.
- [ ] Docker socket mounts are avoided unless strictly required.
- [ ] Linux capabilities are dropped where supported.
- [ ] `no-new-privileges:true` is used where supported.

## Networking

- [ ] Only required ports are exposed.
- [ ] Public exposure risk is documented.
- [ ] Reverse proxy assumptions are documented.
- [ ] Admin interfaces are not exposed directly without authentication.

## Storage

- [ ] Stateful paths or volumes are documented.
- [ ] Backup requirements are documented.
- [ ] Restore notes are included.
- [ ] Host paths avoid machine-specific assumptions unless documented.

## Updates

- [ ] Update process is documented.
- [ ] Breaking upgrade risks are noted where known.
- [ ] Rollback or restore route is documented for stateful apps.

## Review status

Use `metadata.yaml` status values:

- `draft` — added but not reviewed
- `tested` — validated locally
- `recommended` — reviewed and suitable for regular use
- `deprecated` — retained for reference only
