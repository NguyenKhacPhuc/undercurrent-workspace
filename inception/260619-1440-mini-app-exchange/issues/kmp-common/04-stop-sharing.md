---
type: issue
feature: mini-app-exchange
lane: kmp-common
status: ready
wave: 2
estimate: 30m
blocked-by: ["[[02-share-action]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mini-app-exchange
  - status/ready
  - wave/2
---

# [kmp-common] A person can stop sharing a mini-app they shared

**Lane:** kmp-common
**PRD section:** [[PRD#Story 4 — Stop sharing]]
**API contract section:** [[api-contract#`DELETE /v1/mini-apps/share/{shareId}` — stop sharing (revoke)]]

## Why

Gives the sharer control over a link they've changed their mind about. Stopping a share makes the link stop working for new recipients; people who already installed keep their copy.

## Acceptance criteria

- [ ] A mini-app that is currently shared offers a "Stop sharing" action.
- [ ] A mini-app that is not shared does not offer "Stop sharing".
- [ ] After stopping, the mini-app is no longer marked as shared, and the previously generated link no longer resolves to a preview.
- [ ] If stopping fails (offline, server error), the person sees a clear error and the mini-app stays marked as shared (the state stays truthful).

## Blocked by

- [[02-share-action]] — needs the shared state + the kept share identifier to revoke.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`). Consumes `DELETE /v1/mini-apps/share/{id}`; mock until backend [[03-share-revoke]] is deployed.

- **Watch out for:** only flip the local "shared" mark off after the server confirms the revoke.

## Out of scope for this story

- Any reach-back into already-installed copies (there is none, by design).
- A separate "everything I've shared" management screen (out of scope for v1).
