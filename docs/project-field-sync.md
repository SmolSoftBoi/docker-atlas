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
- `Effort`
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

## Token configuration

The workflow uses this token preference:

1. `secrets.PROJECT_SYNC_TOKEN`, if present
2. `GITHUB_TOKEN`, as a fallback

For user-level Projects, `GITHUB_TOKEN` may not have enough access to update Project v2 fields.

If the fallback token cannot access the Project, create a fine-grained personal access token and save it as:

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
- labelled
- unlabelled
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
| `documentation` or `standards` | `Docs` |
| `tooling` | `Tooling` |

### Status

| Condition | Value |
|---|---|
| closed issue | `Done` |
| label `needs review` | `Review` |
| label `blocked` | `Blocked` |
| otherwise open issue | `Triage` |

### Priority

| Label | Value |
|---|---|
| `p0` | `P0` |
| `p1` | `P1` |
| `p2` | `P2` |
| `p3` | `P3` |

### Effort

| Label | Value |
|---|---|
| `effort: small` | `Small` |
| `effort: medium` | `Medium` |
| `effort: large` | `Large` |

### Area

| Label | Value |
|---|---|
| `media` | `Media` |
| `monitoring` | `Monitoring` |
| `automation` | `Automation` |
| `dns` | `DNS` |
| `updates` | `Automation` |
| `proxy` | `Proxy` |
| `notifications` | `Notifications` |
| `remote access` | `Network` |
| `private ai` | `Private AI` |
| `home automation` | `Home Automation` |
| `bookmarks` | `Bookmarks` |
| `storage` | `Storage` |
| `security` | `Security` |
| `documentation` or `standards` | `Docs` |
| `tooling` | `Utility` |

### Risk review

If the Project field is single-select, the workflow chooses the first matching item in this order:

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

## Failure modes

The workflow is designed to fail safely:

- If the Project number is missing, it stops with a clear error.
- If an issue is not in the Project, it logs a warning and skips it.
- If a Project field or option is missing, it logs a warning and continues.
- It does not print tokens or secret values.

## Notes

GitHub Projects field IDs are discovered dynamically from the Project field names, so field names and option names must match the documented values exactly.

If field names are changed in GitHub, update the workflow and this document in the same pull request.
