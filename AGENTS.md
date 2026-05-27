# Docker Atlas Agent Instructions

## Role and Objective
Provide accurate, safe, and reviewable contributions for Docker Atlas, a catalogue-first repository for reusable Docker Compose apps, stacks, standards, and deployment patterns.

## Context
These instructions apply to AI coding agents and automation assistants working in Docker Atlas.

### Project Purpose
Docker Atlas prioritises:
- reusable Compose entries
- clear deployment documentation
- safe defaults
- reviewable changes
- portable paths and environment variables
- explicit security, backup, restore, and update notes

### Repository Structure
Use [ARCHITECTURE.md](ARCHITECTURE.md#repository-model) as the source of truth for the repository model. In short, `apps/` contains single app entries, `stacks/` contains grouped deployment patterns, `shared/` contains reusable Compose fragments, `profiles/` contains host-specific variants, and `templates/`, `standards/`, and `docs/` define reusable guidance. Some catalogue directories may be absent in a fresh checkout until the first matching entry or profile is added.

### Core Standards
Follow these repository standards before adding or changing entries:
- [Compose Style Guide](standards/compose-style-guide.md)
- [Security Checklist](standards/security-checklist.md)
- [Catalogue Standard](docs/catalogue-standard.md)

## Instructions
### Non-Negotiables
- Use `compose.yaml` as the canonical Compose filename.
- Do not add the obsolete top-level `version` property.
- Set a clear top-level Compose [`name`](standards/compose-style-guide.md#project-name) value for each Compose entry.
- Do not commit real secrets, production `.env` files, tokens, passwords, private URLs, or tunnel credentials.
- Keep single app entries in `apps/`.
- Keep grouped deployment patterns in `stacks/`.
- Document ports, volumes, networks, configuration, backup, restore, update, and security risks.
- Prefer safe, portable defaults over host-specific convenience.
- Use small, reviewable changes rather than broad refactors.

### Required Entry Files
Each app or stack entry should include:

```text
compose.yaml
.env.example
metadata.yaml
README.md
```

Use [templates/metadata.yaml](templates/metadata.yaml) and [templates/README.template.md](templates/README.template.md) as the starting point.
See [Catalogue Standard: Required files](docs/catalogue-standard.md#required-files) for the role of each file.

### App vs. Stack Boundaries
- Use `apps/` for one deployable app, even if it has optional integrations; see [Apps](ARCHITECTURE.md#apps).
- Use `stacks/` when multiple services form one operational pattern; see [Stacks](ARCHITECTURE.md#stacks).
- If unsure, start with an app issue and propose a stack separately.

### Security Expectations
Use the [Security Checklist](standards/security-checklist.md) and [Architecture security model](ARCHITECTURE.md#security-model) as the source of truth for risky patterns. Avoid risky permissions by default. If an upstream image requires them, explain why in both `README.md` and `metadata.yaml`.

### Documentation Expectations
Every entry `README.md` should follow [templates/README.template.md](templates/README.template.md) and the [Catalogue Standard review expectations](docs/catalogue-standard.md#review-expectations), including deployment, maintenance, backup, restore, update, and security notes.

### Pull Request Expectations
Every pull request should:
- link the relevant issue
- keep the change focused
- explain any security or deployment risk
- include validation evidence, or state why validation was not run
- update standards or templates when the change introduces a new pattern

### Change Style
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

## Planning and Verification
### Compose Validation
Validate changed Compose files with:

```bash
docker compose -f path/to/compose.yaml config --quiet
```

To validate every Compose file in the repository:

```bash
found=0
while IFS= read -r -d '' file; do
  found=1
  echo "Validating $file"
  docker compose -f "$file" config --quiet || exit 1
done < <(
  for root in apps stacks shared profiles; do
    [ -d "$root" ] && find "$root" -name "compose.yaml" -print0
  done
)

if [ "$found" -eq 0 ]; then
  echo "No compose.yaml files found yet. Skipping validation."
fi
```

If validation cannot be run, state that clearly in the pull request notes.

## Output Format
- Use Markdown only where semantically appropriate, such as lists, code fences, and tables.
- Format file, directory, function, and class names in backticks.
- Preserve exact filenames and paths when referencing repository content.

## Verbosity
- Default to concise summaries.
- For code, use high verbosity with readable names, comments, and straightforward control flow.

## Persistence
- Continue until the user’s query is fully resolved.
- Do not stop on uncertainty; choose the most reasonable path and document assumptions at the end when needed.
- End only when success criteria are met.

## Stop Conditions
Stop and request review before continuing when a change would:
- alter the repository structure
- change the metadata schema
- introduce a new security-sensitive pattern
- require public exposure of an admin service
- remove or weaken existing validation rules
