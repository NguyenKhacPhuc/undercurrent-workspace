---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: ready
wave: 3
estimate: 60m
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

# [backend] `POST /v1/auth/sign-in` authenticates and issues a session

**Lane:** backend
**PRD section:** [[PRD#Story 4 — Sign-in authenticates an existing account]]
**API contract section:** [[api-contract#`POST /v1/auth/sign-in` — exchange credentials for a session]]

## Why

A user who installed the app, signed up, then reinstalled (or just signs in on a second device) needs a way back into their account. This endpoint accepts email + password, verifies them against the stored hash, and hands back a session token.

## Acceptance criteria

- [ ] A request with a known email and the correct password responds 200 with the body shape documented in api-contract: the account object plus a session object.
- [ ] A request with the wrong password OR an unknown email responds 401 with `error.code = "unauthenticated"` AND the same `error.message`, so callers cannot tell from the response which of the two was wrong.
- [ ] A request with missing fields responds 400 with `error.code = "invalid_request"`.
- [ ] Successful sign-in does NOT invalidate other live sessions for the same account; a user signed in on a second device stays signed in on the first.
- [ ] Email matching is case-insensitive (the account stored as `phuc@example.com` matches a sign-in request with `Phuc@Example.com`).
- [ ] Failed sign-in attempts feed the rate limiter from [[09-sign-in-rate-limiting]] (the hook must exist even if the limiter itself lands in a later wave; Construction may stub the hook and wire it for real in story 9). Successful sign-in clears the counter for that email (per [[../../decisions#D6]]).
- [ ] Server logs do not include the plaintext password or any password-hash comparison artifact.
- [ ] Verification of a wrong password is timing-safe (uses the hash library's constant-time comparison, not naive string equality).

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[03-account-record-persisted]] — needs `findByEmail`.
- [[04-session-issued-validated-revoked]] — needs `issueSession`.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Same-message-for-both:** the standard pattern is to always run a hash compare (against a fixed dummy hash if no account exists) so response timing doesn't leak whether the email is registered. Worth keeping unless it complicates things.
- **Watch out for:** the AC says the rate-limit hook must exist. Construction wires the call site; story 9 wires the limiter behind it. Do NOT skip the hook just because the limiter isn't live yet.
- **Watch out for:** session issuance should happen *after* a successful credential check, never before — otherwise a failed sign-in leaves a dangling session row.

## Out of scope for this story

- The rate limiter itself — [[09-sign-in-rate-limiting]].
- Suspending / locking accounts — out of scope entirely for v1.
- Any "remember me" toggle (sessions are uniformly 30-day per [[../../decisions#D3]]).
