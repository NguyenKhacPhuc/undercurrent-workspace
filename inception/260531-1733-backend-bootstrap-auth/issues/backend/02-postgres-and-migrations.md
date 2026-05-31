---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: done
claimed-by: NguyenKhacPhuc
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent-backend/pull/2
merged-at: 2026-05-31T12:04:48Z
wave: 1
estimate: 60m
blocked-by:
  - "[[01-ktor-deploys-to-railway-with-health]]"
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/done
  - wave/1
---

# [backend] A persistent store is wired with migrations

**Lane:** backend
**PRD section:** [[PRD#Story 2 — A persistent store is wired]]
**API contract section:** n/a (no new endpoints)

## Why

The Account and Session stories that follow need somewhere durable to write to. This story attaches a Postgres database to the deployed app, wires a migration tool, and proves that a no-op migration runs on startup without re-applying itself on subsequent restarts. After this lands, every later story can author a migration and assume it will be applied.

## Acceptance criteria

- [ ] A managed Postgres instance is attached to the deployed environment.
- [ ] The app boots successfully against the attached database; failing to connect is a fatal startup error (the app does not silently start with no DB).
- [ ] On startup, any pending migrations run and are recorded as applied; restarting the app does not re-run them.
- [ ] At least one trivial baseline migration exists (e.g. a no-op or a simple "schema initialized" marker) so the migration mechanism is observably exercised.
- [ ] The DB connection settings come from environment variables (no hardcoded credentials, no committed connection strings).
- [ ] `backend/CLAUDE.md` documents how to run a Postgres locally for development (Docker command or equivalent) and how to point the app at it.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[01-ktor-deploys-to-railway-with-health]] — there has to be a running app to attach a database to.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Postgres provisioning:** Railway's one-click Postgres add-on. The connection URL drops into env vars automatically.
- **Migration tool:** Construction's choice. Flyway and Liquibase both have decent Ktor stories; a hand-rolled "list of SQL files + a `schema_migrations` table" also works at this scale.
- **Watch out for:** keep the migration step inline with app startup for v1 simplicity, but make it skippable via a `MIGRATIONS_ENABLED=false` toggle so future "run migrations as a one-off Railway job" can land without rework.

## Out of scope for this story

- Any actual schema for accounts or sessions — [[03-account-record-persisted]] and [[04-session-issued-validated-revoked]] own those.
- A purge job for expired sessions — see [[../../out-of-scope]].
- DB-level backup / restore — outside Inception scope; Railway's defaults are accepted.
