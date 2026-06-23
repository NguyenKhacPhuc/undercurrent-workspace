---
type: issue
feature: mini-app-exchange
lane: kmp-common
status: ready
wave: 1
estimate: 75m
blocked-by: ["[[01-shareable-foundation]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mini-app-exchange
  - status/ready
  - wave/1
---

# [kmp-common] A person can preview a shared mini-app and install it

**Lane:** kmp-common
**PRD section:** [[PRD#Story 2 — Preview a shared mini-app]], [[PRD#Story 3 — Install from a preview]]
**API contract section:** [[api-contract#`GET /v1/mini-apps/share/{shareId}` — fetch a shared mini-app for preview/install]]

## Why

The recipient's moment of trust: given a share identifier, the app fetches what was shared and shows a preview — name, who shared it, what it can do, and a clear "this author isn't vetted" framing — then lets the person install it. This is the screen the per-platform deep links route into.

## Acceptance criteria

- [ ] Given a share identifier, the app shows a preview with the mini-app's name, icon, and description (if any).
- [ ] The preview shows who shared it (the sharer's display name).
- [ ] The preview lists every capability the mini-app will be able to request — and only the capabilities this app actually offers (any the app does not offer are not shown and cannot be granted).
- [ ] The preview makes clear the author is not vetted by Undercurrent and the person is choosing to trust them.
- [ ] If the share is unknown or has been stopped, the preview shows a clear "this mini-app is no longer available" message and offers no install.
- [ ] If the fetch fails for a connectivity reason, the person sees a recoverable error (distinct from "no longer available") and can retry.
- [ ] Installing from the preview adds the mini-app to the person's Mini Apps list without requiring an account.
- [ ] The first launch of the installed mini-app runs the existing capability-approval prompt; nothing the mini-app declared runs before the person approves.
- [ ] The installed mini-app records that it came from a shared link and who shared it.

## Blocked by

- [[01-shareable-foundation]] — needs install-from-bundle + the capability clamp.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`). Consumes `GET /v1/mini-apps/share/{id}`; build against a mock until backend [[02-share-fetch]] is deployed.

- **Existing pattern to mirror:** the existing first-run capability-consent prompt for HTML mini-apps — the install path must reuse it, not duplicate it. The capability clamp from [[01-shareable-foundation]] drives the "only what this app offers" list.
- **Watch out for:** keep "no longer available" (a 404 from the server) visually and behaviorally distinct from a transient network error — only the latter offers retry.

## Out of scope for this story

- How the preview gets opened from outside the app (per-platform deep-link stories).
- Sharing ([[02-share-action]]) and revoke ([[04-stop-sharing]]).
