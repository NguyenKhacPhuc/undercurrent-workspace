---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 2
estimate: 60m
blocked-by:
  - "[[02-postgres-and-migrations]]"
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/in-progress
  - wave/2
---

# [backend] A session can be issued, validated by middleware, and revoked

**Lane:** backend
**PRD section:** Foundation for [[PRD#Story 3 — Sign-up creates an account]], [[PRD#Story 4 — Sign-in authenticates an existing account]], [[PRD#Story 5 — The BE can identify the signed-in user]], [[PRD#Story 6 — Sign-out invalidates a session]]
**API contract section:** [[api-contract#Conventions]] (Bearer-token convention)

## Why

Four of the five endpoint stories depend on session machinery: sign-up issues one, sign-in issues one, get-current-user validates one, sign-out revokes one. Landing this as a foundation slice means each of those endpoint stories only handles its endpoint-specific concern, and the session shape stays consistent across all of them.

## Acceptance criteria

Foundation slice — no HTTP endpoints yet. The contract is observable through the session module's public surface plus a migration applied to the DB.

- [ ] A migration creates a sessions table holding: opaque token (or its hash), the account id it grants access to, an issued-at timestamp, an expires-at timestamp, and an optional revoked-at timestamp.
- [ ] Issuing a session for an account id returns a token a caller can hand to a client; the same token, when later presented, resolves back to the same account id.
- [ ] A session is valid only when (a) it exists in the table, (b) its expires-at is in the future, and (c) its revoked-at is unset.
- [ ] Validation of an invalid, expired, revoked, or unknown token returns an observably-unauthenticated result (distinguishable from "valid token for some specific account").
- [ ] Revoking a session is observable: the same token, validated after revocation, no longer authenticates.
- [ ] Revoking one session does not affect any other session of the same account.
- [ ] The HTTP middleware that reads `Authorization: Bearer <token>` and gates downstream handlers is wired and works against the validation surface above.
- [ ] Default session TTL is 30 days from issuance; this number lives in one place so future Inceptions (or [[../../open-questions#Q4]]) can change it.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[02-postgres-and-migrations]] — needs DB + migrations.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Token shape:** opaque + server-stored per [[../../decisions#D3]]. Construction picks the source of entropy and the at-rest representation (e.g. store SHA-256 of the token so a DB leak doesn't hand attackers usable tokens).
- **Middleware shape:** a request that requires auth and arrives without a valid bearer should short-circuit to the standard 401 error envelope from [[../../api-contract#Conventions]].
- **Watch out for:** "sign-out is idempotent" — the revoke operation must accept "no session matched" without erroring, because the sign-out endpoint never returns 4xx (see [[08-sign-out-endpoint]]).
- **Watch out for:** do NOT add a "refresh token" concept here. It is explicitly out of scope per [[../../decisions#D3]].

## Out of scope for this story

- Sign-up / sign-in endpoint logic (those just *call* `issueSession`).
- A janitor / sweep job for expired sessions — see [[../../out-of-scope]].
- Rate limiting — [[09-sign-in-rate-limiting]].
