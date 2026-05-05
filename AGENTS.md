# Role and Objective
Provide accurate, safe, and reviewable contributions for Docker Atlas, a catalogue-first repository for reusable Docker Compose apps, stacks, standards, and deployment patterns.

# Context
These instructions apply to AI coding agents and automation assistants working in Docker Atlas.

## Project Purpose
Docker Atlas prioritises:
- reusable Compose entries
- clear deployment documentation
- safe defaults
- reviewable changes
- portable paths and environment variables
- explicit security, backup, restore, and update notes

## Repository Structure
- `apps/` — single application entries
- `stacks/` — grouped multi-service deployment patterns
- `shared/` — shared Compose fragments, common networks, and reusable patterns
- `profiles/` — host-specific or environment-specific overrides
- `templates/` — reusable metadata and README templates
- `standards/` — style, security, and catalogue rules
- `docs/` — supporting documentation and design notes
- `.github/` — issue templates, pull request template, and validation workflows

## Core Standards
Follow these repository standards before adding or changing entries:
- `standards/compose-style-guide.md`
- `standards/security-checklist.md`
- `docs/catalogue-standard.md`

# Instructions
## Non-Negotiables
- Use `compose.yaml` as the canonical Compose filename.
- Do not add the obsolete top-level `version` property.
- Do not commit real secrets, production `.env` files, tokens, passwords, private URLs, or tunnel credentials.
- Keep single app entries in `apps/`.
- Keep grouped deployment patterns in `stacks/`.
- Document ports, volumes, networks, configuration, backup, restore, update, and security risks.
- Prefer safe, portable defaults over host-specific convenience.
- Use small, reviewable changes rather than broad refactors.

## Required Entry Files
Each app or stack entry should include:

```text
compose.yaml
.env.example
metadata.yaml
README.md
```

Use `templates/metadata.yaml` and `templates/README.template.md` as the starting point.

## App vs. Stack Boundaries
- Use `apps/` for one deployable app, even if it has optional integrations.
- Use `stacks/` when multiple services form one operational pattern, such as:
  - n8n + Postgres + Redis
  - Jellyfin or Plex + Sonarr + Radarr + Prowlarr + qBittorrent
  - Open WebUI + optional n8n + optional local model endpoint
- If unsure, start with an app issue and propose a stack separately.

## Security Expectations
Treat these as high-risk and document them clearly when required:
- Docker socket mounts
- `privileged: true`
- `network_mode: host`
- public admin web UIs
- default credentials
- persistent sensitive data
- tunnel or reverse proxy exposure
- hardware/device mounts

Avoid risky permissions by default. If an upstream image requires them, explain why in both `README.md` and `metadata.yaml`.

## Documentation Expectations
Every entry `README.md` should explain:
- what the app or stack does
- what services are included
- required ports
- required volumes and paths
- environment variables
- first-run setup
- backup and restore
- update steps
- security notes
- reverse proxy assumptions, where relevant

## Pull Request Expectations
Every pull request should:
- link the relevant issue
- keep the change focused
- explain any security or deployment risk
- include validation evidence, or state why validation was not run
- update standards or templates when the change introduces a new pattern

## Change Style
Prefer:
- explicit defaults
- clear comments where needed
- portable environment variables such as `TZ` and `DATA_ROOT`
- predictable folder names in lowercase kebab-case
- additive changes that do not break existing entries

Avoid:
- broad reorganisation without an issue
- unreviewed host-specific assumptions
- unexplained privileged access
- hidden behaviour in scripts
- committing generated files unless the repo standard requires them

# Planning and Verification
## Compose Validation
Validate changed Compose files with:

```bash
docker compose -f path/to/compose.yaml config --quiet
```

To validate every Compose file in the repository:

```bash
find apps stacks shared profiles -name "compose.yaml" -print0 | while IFS= read -r -d '' file; do
  echo "Validating $file"
  docker compose -f "$file" config --quiet
done
```

If validation cannot be run, state that clearly in the pull request notes.

# Output Format
- Use Markdown only where semantically appropriate, such as lists, code fences, and tables.
- Format file, directory, function, and class names in backticks.
- Preserve exact filenames and paths when referencing repository content.

# Verbosity
- Default to concise summaries.
- For code, use high verbosity with readable names, comments, and straightforward control flow.

# Persistence
- Continue until the user’s query is fully resolved.
- Do not stop on uncertainty; choose the most reasonable path and document assumptions at the end when needed.
- End only when success criteria are met.

# Stop Conditions
Stop and request review before continuing when a change would:
- alter the repository structure
- change the metadata schema
- introduce a new security-sensitive pattern
- require public exposure of an admin service
- remove or weaken existing validation rules
