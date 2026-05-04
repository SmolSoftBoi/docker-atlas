# Docker Atlas change

## Summary

- 

## Linked issue

Closes #

## Type of change

- [ ] New app entry
- [ ] New stack entry
- [ ] Standards or documentation update
- [ ] Fix existing entry
- [ ] Tooling or validation update
- [ ] Issue or pull request template update

## Entry details

Complete this section for app or stack entries.

| Field | Value |
|---|---|
| Entry path |  |
| Category |  |
| Status | draft / tested / recommended / deprecated |
| Internet exposure | none / LAN only / reverse proxy / public |
| Stateful | yes / no |
| Docker socket required | yes / no |
| Host networking required | yes / no |
| Privileged mode required | yes / no |

## Validation

- [ ] I ran `docker compose config --quiet` for every changed `compose.yaml`
- [ ] I checked the Compose file does not include the obsolete top-level `version` property
- [ ] I checked all required environment variables are documented
- [ ] I checked no real `.env` files, secrets, tokens, passwords, or private URLs are committed

## Catalogue checklist

- [ ] Uses `compose.yaml`
- [ ] Includes `.env.example` where configuration is needed
- [ ] Includes `metadata.yaml`
- [ ] Includes README deployment notes
- [ ] Documents ports
- [ ] Documents volumes and backup requirements
- [ ] Documents restore steps for stateful apps
- [ ] Documents update steps
- [ ] Documents security risks and internet exposure
- [ ] Documents reverse proxy assumptions where relevant

## Security checklist

- [ ] Docker socket access is avoided or clearly justified
- [ ] Host networking is avoided or clearly justified
- [ ] Privileged mode is avoided or clearly justified
- [ ] Default credentials are changed or clearly flagged
- [ ] Admin interfaces are not exposed directly without guidance

## Notes

Add review notes, known limitations, screenshots, validation output, or follow-up tasks here.
