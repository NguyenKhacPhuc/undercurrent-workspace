---
type: issue
feature: mini-app-exchange
lane: kmp-common
status: ready
wave: 1
estimate: 60m
blocked-by: ["[[01-shareable-foundation]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mini-app-exchange
  - status/ready
  - wave/1
---

# [kmp-common] A person can share an HTML mini-app as a link and QR

**Lane:** kmp-common
**PRD section:** [[PRD#Story 1 — Share a mini-app]]
**API contract section:** [[api-contract#`POST /v1/mini-apps/share` — create a share link for a mini-app]]

## Why

This is the sharer's side of the loop: from one of their HTML mini-apps, the person turns it into a link and a scannable QR they can hand to anyone.

## Acceptance criteria

- [ ] Each HTML mini-app offers a "Share" action.
- [ ] A native (trigger-prompt-only) mini-app does not offer the Share action.
- [ ] Choosing Share while signed out prompts the person to sign in first; sharing does not proceed until they are signed in.
- [ ] Choosing Share while signed in produces a link and a scannable QR code for that mini-app.
- [ ] The link can be sent onward through the system share sheet.
- [ ] After a successful share, the mini-app is visibly marked as shared.
- [ ] If sharing fails (offline, server error), the person sees a clear, recoverable error and the mini-app is not falsely marked as shared.

## Blocked by

- [[01-shareable-foundation]] — needs a mini-app to be able to remember it was shared.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`). Consumes `POST /v1/mini-apps/share`; build against a mock until backend [[01-share-create]] is deployed.

- **Existing pattern to mirror:** the existing system-share capability already in the substrate (the `share` offerable action / share-content path) — reuse it for the share sheet rather than adding platform share code here.
- **QR rendering** turns the deep link into an on-screen code. The *behavior* (show a QR) is shared; the image primitive differs per platform, so a thin platform-provided renderer behind a common interface is expected — keep the rest in commonMain.
- **Watch out for:** only mark the mini-app shared after the server confirms; keep the returned share identifier so [[04-stop-sharing]] can revoke it.

## Out of scope for this story

- The recipient-side preview/install ([[03-install-preview]]) and revoke ([[04-stop-sharing]]).
- Deep-link registration (per-platform stories).
