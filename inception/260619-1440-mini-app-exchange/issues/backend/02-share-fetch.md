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

# [Backend] Anyone with a share link can fetch the mini-app to preview and install it

**Lane:** Backend
**PRD section:** [[PRD#Story 2 — Preview a shared mini-app]]
**API contract section:** [[api-contract#`GET /v1/mini-apps/share/{shareId}` — fetch a shared mini-app for preview/install]]

## Why

A recipient who opens a share link needs to pull down what was shared — the name, icon, description, the HTML, the declared capabilities, and who shared it — so the app can show a preview and, on approval, install it. No account required, because receiving must be friction-free.

## Acceptance criteria

- [ ] Anyone (no sign-in) can fetch a shared mini-app by its share identifier and receive its name, icon, description, HTML document, declared capabilities, the sharer's display name, and when it was shared.
- [ ] Fetching an identifier that never existed returns a "not available" response.
- [ ] Fetching an identifier whose share has been stopped returns the same "not available" response — a revoked share is indistinguishable from one that never existed.

## Blocked by

- [[01-share-create]] — needs the share-record store and identifier format.

## Hints (non-binding)

- **Watch out for:** revoked and unknown must both be 404 with the same body — no signal about whether an id ever existed. Read-only; no auth header required.

## Out of scope for this story

- The act of installing (that's client-side) and revoking ([[03-share-revoke]]).
