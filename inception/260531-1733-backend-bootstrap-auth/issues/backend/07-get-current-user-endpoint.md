---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: ready
wave: 3
estimate: 30m
blocked-by:
  - "[[03-account-record-persisted]]"
  - "[[04-session-issued-validated-revoked]]"
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/ready
  - wave/3
---

# [backend] `GET /v1/me` returns the signed-in account

**Lane:** backend
**PRD section:** [[PRD#Story 5 — The BE can identify the signed-in user]]
**API contract section:** [[api-contract#`GET /v1/me` — return the signed-in account]]

## Why

Mobile clients need a way to confirm "yes, I'm still signed in, and here's who I am" after a fresh app launch or a long idle period. This endpoint is the simplest test of the session-middleware pipeline: present a token, get the account back.

## Acceptance criteria

- [ ] A request with a valid session token responds 200 with the body shape documented in api-contract: `{ "account": { id, displayName, email, createdAtMs } }`.
- [ ] A request with no `Authorization` header responds 401 with `error.code = "unauthenticated"`.
- [ ] A request with an unknown, expired, or revoked session token responds 401 with `error.code = "unauthenticated"`.
- [ ] The response never includes the password hash or any other field beyond what the api-contract documents.
- [ ] The response has `Cache-Control: private, no-store` (or equivalent) so caches don't fan out user identity.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[03-account-record-persisted]] — needs `findById`.
- [[04-session-issued-validated-revoked]] — needs the auth middleware.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Endpoint shape:** the simplest possible authed read. Most of the work is making sure the middleware composes cleanly so future authed endpoints (post-v1) inherit the same protection automatically.
- **Watch out for:** treat "session valid but the account it points at has been deleted" as an unauthenticated state, even though the spec for v1 says accounts can't be deleted. Defensive; the assumption may not hold forever.

## Out of scope for this story

- `PATCH /v1/me` — see [[../../out-of-scope]].
- Avatar / extended profile fields — see [[../../out-of-scope]].
