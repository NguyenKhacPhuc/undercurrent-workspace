---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 3
estimate: 30m
blocked-by:
  - "[[04-session-issued-validated-revoked]]"
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/in-progress
  - wave/3
---

# [backend] `POST /v1/auth/sign-out` invalidates the presented session

**Lane:** backend
**PRD section:** [[PRD#Story 6 — Sign-out invalidates a session]]
**API contract section:** [[api-contract#`POST /v1/auth/sign-out` — invalidate the presented session]]

## Why

A user who hands their device to someone else, or who loses a device, needs a way to revoke that session. Sign-out is the bare minimum for that — revoke the specific token, leave other sessions alone.

## Acceptance criteria

- [ ] A request with a valid session token responds 204 and invalidates that exact token; a subsequent authenticated request reusing the same token responds 401.
- [ ] A request with a missing, unknown, expired, or already-invalidated token also responds 204 — the endpoint is idempotent and leaks no information about which case it was.
- [ ] Sign-out does NOT invalidate any other live session of the same account.
- [ ] After a successful sign-out, the revoked session is observably distinguishable from "never existed" at the persistence layer (`revoked_at` is set, not row-deleted) so a future audit story can reason about it. Construction picks whether to delete or soft-revoke; either is acceptable as long as validation rejects it.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[04-session-issued-validated-revoked]] — needs the `revokeSession` surface and the bearer-token reader (since the handler must read the token from the request even though it doesn't go through the gating middleware).

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Always 204:** that's the surprising part of this story. Even "no Authorization header at all" responds 204. Per [[../../api-contract#`POST /v1/auth/sign-out` — invalidate the presented session]].
- **Watch out for:** this endpoint does NOT use the same middleware as `/v1/me`. The middleware short-circuits to 401 on bad tokens; sign-out cannot. Construction wires sign-out outside that middleware (or with a permissive variant).

## Out of scope for this story

- "Sign out everywhere" / revoke all sessions of an account — out of scope for v1 (could be a future endpoint once we surface session management to users).
