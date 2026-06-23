---
type: issue
feature: mini-app-exchange
lane: backend
status: ready
wave: 1
estimate: 30m
blocked-by: ["[[01-share-create]]"]
tags:
  - inception/issue
  - lane/backend
  - feature/mini-app-exchange
  - status/ready
  - wave/1
---

# [Backend] The person who shared a mini-app can stop sharing it

**Lane:** Backend
**PRD section:** [[PRD#Story 4 — Stop sharing]]
**API contract section:** [[api-contract#`DELETE /v1/mini-apps/share/{shareId}` — stop sharing (revoke)]]

## Why

A sharer needs a way to take back a link they regret. Stopping a share makes the link stop resolving for new recipients, while leaving anyone who already installed unaffected (their copy lives on their own device).

## Acceptance criteria

- [ ] The owner of a share can stop it; afterward fetching that link returns "not available".
- [ ] A caller who is not signed in cannot stop a share.
- [ ] A signed-in caller who is not the owner of the share cannot stop it and is refused.
- [ ] Stopping a share that does not exist (or was already stopped) is handled cleanly, not with a server error.

## Blocked by

- [[01-share-create]] — needs the share-record store + ownership.

## Hints (non-binding)

- **Likely work:** mark the record revoked (a soft `revokedAtMs`) rather than hard-deleting — keeps fetch's 404-on-revoke cheap and auditable.
- **Watch out for:** ownership check is the security gate here — only the creating account may revoke.

## Out of scope for this story

- Any effect on already-installed copies (there is none, by design).
