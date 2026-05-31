---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 2
estimate: 45m
blocked-by:
  - "[[02-postgres-and-migrations]]"
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/in-progress
  - wave/2
---

# [backend] An account record can be persisted, fetched by id, fetched by email

**Lane:** backend
**PRD section:** Foundation for [[PRD#Story 3 — Sign-up creates an account]] and [[PRD#Story 4 — Sign-in authenticates an existing account]]
**API contract section:** n/a (no new endpoints — schema only)

## Why

Sign-up needs to write an account; sign-in needs to look one up by email; get-current-user needs to look one up by id. All three stories need the same persistence shape and it is cleaner to land the schema + the read/write contract once than to inline it three times.

## Acceptance criteria

This is a foundation slice — no HTTP endpoints yet. The contract is observable through the persistence layer's public surface and through a migration applied to the DB.

- [ ] A migration creates an accounts table holding: stable id, email (unique, stored lowercased), display name, password hash, created-at timestamp.
- [ ] Inserting an account is observable: a subsequent fetch by id returns the same display name, email, and created-at.
- [ ] Fetching by email is case-insensitive (`Phuc@Example.com` returns the account stored as `phuc@example.com`).
- [ ] Fetching by an unknown id or unknown email returns an observably absent result (distinguishable from any "real" account, including one whose fields are empty strings).
- [ ] Attempting to insert two accounts with the same normalized email fails at the persistence layer with a clear conflict signal (the calling code can branch on it without parsing exception messages).
- [ ] The password hash field is opaque to the persistence layer — it just stores and returns whatever bytes Construction passes in. Hashing/verification is the caller's concern (lives in the sign-up / sign-in stories).

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[02-postgres-and-migrations]] — needs both the connected DB and the migration runner.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Id shape:** `acct.<uuid12>` per [[../../../../CONTEXT#Account]]. Construction picks how to generate it.
- **Email normalization:** lowercase + trim before any uniqueness check or insert. Apply at the persistence-layer boundary so callers don't have to remember.
- **Watch out for:** the unique-index conflict needs to surface as a typed signal, not a stringly-typed "SQLState X23505" check. Sign-up will translate it to a 409 — see [[05-sign-up-endpoint]].
- **Watch out for:** keep this layer free of password-hashing logic. The unit test for hashing should not require a DB.

## Out of scope for this story

- Sign-up endpoint logic — [[05-sign-up-endpoint]].
- Password hashing — folded into [[05-sign-up-endpoint]] and [[06-sign-in-endpoint]].
- Updating accounts (`PATCH /v1/me`) — see [[../../out-of-scope]].
- Deleting accounts — see [[../../out-of-scope]].
