---
type: issue
feature: mini-app-exchange
lane: backend
status: ready
wave: 0
estimate: 60m
blocked-by: []
tags:
  - inception/issue
  - lane/backend
  - feature/mini-app-exchange
  - status/ready
  - wave/0
---

# [Backend] A signed-in person can mint a share link for a mini-app

**Lane:** Backend
**PRD section:** [[PRD#Story 1 — Share a mini-app]]
**API contract section:** [[api-contract#`POST /v1/mini-apps/share` — create a share link for a mini-app]]

## Why

The whole feature hangs off being able to hand a mini-app to someone else. This is the foundation: a signed-in person sends the assistant-built mini-app to the backend, which stores it and returns a link and QR-ready deep link pointing at it. Without this, nothing can be shared.

## Acceptance criteria

- [ ] A signed-in caller can submit a mini-app (its name, icon, optional description, its HTML document, and the capabilities it declares) and get back a link, a deep link, and an identifier for the share.
- [ ] The stored share remembers who created it and the creator's display name at the time of sharing.
- [ ] A caller who is not signed in cannot create a share and gets an "unauthenticated" response.
- [ ] A submission missing a name or its HTML document is rejected with a clear "invalid request" response.
- [ ] A submission whose HTML document exceeds the size cap is rejected with a "too large" response rather than stored.
- [ ] Each successful submission produces a fresh, unguessable share identifier (sharing the same mini-app twice yields two distinct links).

## Blocked by

- nothing — independently grabbable.

## Hints (non-binding)

> [!tip]
> Construction reads `backend/CLAUDE.md` for build/test/deploy. Mirror the existing auth endpoints for envelope shape, bearer-token validation, and error codes.

- **Existing pattern to mirror:** the auth sign-up/sign-in handlers + their `BaseResponse<T>` wrapping and `{code,message,details}` errors (`inception/260531-1733-backend-bootstrap-auth/`).
- **Likely work:** a new Postgres table for share records (bundle fields + owner account id + display-name snapshot + created/revoked timestamps), and the `POST` route. Identifier is the short `share.<base62-10>` form from the contract.
- **Watch out for:** the display-name snapshot — store it at share time so attribution survives a later rename. Size cap is **[DRIVER GUESS 256 KB — see [[open-questions#Q1]]]**.

## Out of scope for this story

- Fetching (separate story [[02-share-fetch]]) and revoking ([[03-share-revoke]]).
- Listing a user's shares; any moderation; any rate limiting beyond what already exists.
