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

# [backend] `POST /v1/auth/sign-up` registers an account and issues a session

**Lane:** backend
**PRD section:** [[PRD#Story 3 — Sign-up creates an account]]
**API contract section:** [[api-contract#`POST /v1/auth/sign-up` — register a new account]]

## Why

This is the first user-facing BE endpoint. After it ships, a client (curl in Construction, mobile in a later Inception) can create a real account and walk away with a session token usable on every other authenticated endpoint.

## Acceptance criteria

- [ ] A request with a valid display name (non-empty after trim; ≤ 40 chars; unicode allowed), valid email (contains `@` with a `.` after it, no spaces, case-insensitive), and valid password (≥ 8 chars, no max, no required character classes) creates an account row and a session row and responds with status 201 and the body shape documented in api-contract. Validation rules per [[../../decisions#D2]] + the Q2 resolution in [[../../open-questions#Q2 — Exact validation rules for email, password, displayName — 2026-05-31]].
- [ ] The stored account holds a salted hash of the password, never the password itself; the response body never echoes the password.
- [ ] Two requests with the same email — sequential or concurrent — produce exactly one account; the loser responds 409 with `error.code = "email_already_registered"` and no session is issued.
- [ ] A request with an invalid field (e.g. malformed email, empty display name, too-short password) responds 400 with `error.code = "invalid_request"` and no account is created.
- [ ] The session token in the response is immediately usable on a subsequent authenticated request (validated by the middleware from [[04-session-issued-validated-revoked]]).
- [ ] Server logs do not include the plaintext password or the password hash.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[03-account-record-persisted]] — needs the accounts table + persistence.
- [[04-session-issued-validated-revoked]] — needs `issueSession`.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Hashing:** argon2id with sensible defaults is the modern recommendation; Construction confirms based on what's available in the chosen JVM library. Whatever's picked, the parameters live in one named place so we can rotate later.
- **Validation rules:** ratified — see the Q2 resolution in [[../../open-questions]] (displayName ≤ 40 chars trimmed unicode; email loose format + case-insensitive; password ≥ 8 chars, no max, no classes).
- **Watch out for:** the concurrent-duplicate-signup case. Two requests racing on the same email must converge to "exactly one account exists" — rely on the unique index from [[03-account-record-persisted]], catch the typed conflict signal, and respond 409 cleanly.
- **Watch out for:** ordering — write the account first, then issue the session. If session-issue fails after the account is written, the user is in a valid sign-in-able state; that's recoverable.

## Out of scope for this story

- Email verification — see [[../../out-of-scope]].
- Welcome email — see [[../../out-of-scope]].
- Any client-side wiring — see [[../../decisions#D4]].
