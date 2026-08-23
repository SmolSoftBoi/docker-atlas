# Project field sync automation

Docker Atlas uses a GitHub Project named `Docker Atlas Catalogue` to track catalogue work without overloading issues with labels.

The workflow in `.github/workflows/sync-project-fields.yml` syncs issue labels, state, and milestones into Project fields.

## Workflow

```text
.github/workflows/sync-project-fields.yml
```

## What it updates

The workflow maps issue metadata into these Project fields:

- `Entry type`
- `Status`
- `Priority`
- `Size`
- `Area`
- `Risk review`
- `Implementation wave`

It intentionally does not update checklist-style fields such as `Upstream verified` or `Compose validated`, because those should reflect implementation evidence.

## Required repository variables

Create these repository variables under:

```text
Settings → Secrets and variables → Actions → Variables
```

| Variable | Required | Example | Purpose |
|---|---:|---|---|
| `DOCKER_ATLAS_PROJECT_NUMBER` | Yes | `1` | The visible Project number from the Project URL. |
| `DOCKER_ATLAS_PROJECT_OWNER` | No | `SmolSoftBoi` | Project owner. Defaults to the repository owner if omitted. |

## Required Project schema

Before enabling the workflow, make sure the Project contains these fields and option names. Field and option matching is case-insensitive, but otherwise exact.

| Field | Type | Required options |
|---|---|---|
| `Status` | Single select | `Backlog`, `Triage`, `Ready`, `In progress`, `In review`, `Done`, `Blocked` |
| `Entry type` | Single select | `App`, `Stack`, `Documentation`, `Tooling` |
| `Area` | Single select | `Media`, `Monitoring`, `Automation`, `Network`, `DNS`, `Proxy`, `Notifications`, `Private AI`, `Home Automation`, `Bookmarks`, `Storage`, `Security`, `Documentation`, `Utility` |
| `Priority` | Single select | `P0`, `P1`, `P2`, `P3` |
| `Size` | Single select | `XS`, `S`, `M`, `L`, `XL` |
| `Risk review` | Single select or multi-select | `None`, `Security`, `Public exposure`, `Secrets`, `Docker socket`, `Network`, `Storage`, `Hardware` |
| `Implementation wave` | Single select | `Foundation`, `MVP`, `Media Core`, `Media Expansion`, `Stacks`, `Later` |

The workflow does not create or rename Project fields. Add the `Risk review` field before enabling the workflow if it is not already present.

## Token configuration

The workflow requires a project-capable token because the repository-scoped `GITHUB_TOKEN` cannot access Projects v2. Create a personal access token and save it as the repository secret:

```text
PROJECT_SYNC_TOKEN
```

Recommended token access:

- access to the `SmolSoftBoi/docker-atlas` repository
- read access to issues
- write access to Projects

Keep token permissions as narrow as GitHub allows.

## Supported triggers

The workflow runs when an issue is:

- opened
- edited
- labeled
- unlabeled
- milestoned
- demilestoned
- reopened
- closed

It can also be run manually with `workflow_dispatch`.

## Manual sync

Use manual dispatch from:

```text
Actions → Sync project fields → Run workflow
```

Inputs:

| Input | Use |
|---|---|
| `issue_number` | Sync one issue. Leave blank to sync all open issues. |
| `dry_run` | Log planned updates without changing Project fields. |

Recommended first run:

```text
issue_number: blank
dry_run: true
```

If the dry run looks correct, run again with:

```text
dry_run: false
```

## Label mappings

### Entry type

| Label or title | Value |
|---|---|
| `app request` or title starts `Add app:` | `App` |
| `stack request` or title starts `Add stack:` | `Stack` |
| `documentation` or `standards` | `Documentation` |
| `tooling` | `Tooling` |

### Status

| Condition | Value |
|---|---|
| closed issue | `Done` |
| label `needs review` | `In review` |
| label `blocked` | `Blocked` |
| label `needs triage` | `Triage` |
| newly opened or reopened issue | `Triage` |
| no matching condition | leave the existing Project status unchanged |

### Priority

| Label | Value |
|---|---|
| `p0` | `P0` |
| `p1` | `P1` |
| `p2` | `P2` |
| `p3` | `P3` |

### Size

Effort labels map to the Project's existing `Size` field.

| Label | Value |
|---|---|
| `effort: small` | `S` |
| `effort: medium` | `M` |
| `effort: large` | `L` |

### Area

| Label | Value |
|---|---|
| `media` | `Media` |
| `monitoring` | `Monitoring` |
| `private ai` | `Private AI` |
| `home automation` | `Home Automation` |
| `automation` | `Automation` |
| `dns` | `DNS` |
| `updates` | `Utility` |
| `bookmarks` | `Bookmarks` |
| `storage` | `Storage` |
| `docs`, `documents`, `documentation`, or `standards` | `Documentation` |
| `tooling`, `utility`, or `travel` | `Utility` |
| `security` | `Security` |
| `proxy` | `Proxy` |
| `remote access` | `Network` |
| `notifications` | `Notifications` |

When several Area labels match, the first matching row in this table wins.

### Risk review

For a single-select field, the workflow chooses the first matching item in this order. For a multi-select field, it applies every unique matching value.

| Label | Value |
|---|---|
| `risk: docker socket` | `Docker socket` |
| `risk: privileged` | `Security` |
| `risk: host network` | `Network` |
| `risk: public exposure` | `Public exposure` |
| `risk: secrets` | `Secrets` |
| `risk: network impact` | `Network` |
| `risk: hardware access` | `Hardware` |
| `risk: stateful` | `Storage` |
| no matching risk label | `None` |

## Implementation wave mapping

The workflow maps the issue milestone title into `Implementation wave`.

| Milestone title contains | Value |
|---|---|
| `Foundation` or starts with `M0` | `Foundation` |
| `MVP` or starts with `M1` | `MVP` |
| `Media Core` or starts with `M2` | `Media Core` |
| `Media Expansion` or starts with `M3` | `Media Expansion` |
| `Stack` or starts with `M4` | `Stacks` |
| anything else | `Later` |

When a label or milestone no longer maps to an optional field value, the workflow clears the corresponding Project field instead of leaving stale metadata in place. `Status` is preserved when no explicit status condition applies.

## Failure modes

The workflow is designed to fail safely:

- If the Project number is missing, it stops with a clear error.
- For a newly opened issue, it retries Project lookup for up to 20 seconds while the built-in auto-add workflow runs.
- If an issue is still not in the Project, it logs a warning and skips it.
- If a Project field or option is missing, it logs a warning and continues.
- It does not print tokens or secret values.

## Notes

GitHub Projects field IDs are discovered dynamically from the Project field names, so field names and option names must match the documented values exactly.

If field names are changed in GitHub, update the workflow and this document in the same pull request.
