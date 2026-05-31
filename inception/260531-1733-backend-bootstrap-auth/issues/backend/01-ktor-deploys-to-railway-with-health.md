---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: done
claimed-by: NguyenKhacPhuc
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent-backend/pull/1
merged-at: 2026-05-31T11:22:07Z
wave: 0
estimate: 90m
blocked-by: []
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/done
  - wave/0
---

# [backend] A backend service is deployed and reachable

**Lane:** backend
**PRD section:** [[PRD#Story 1 — There is a deployed backend]]
**API contract section:** [[api-contract#`GET /health` — liveness probe]]

## Why

Every other story in this Inception writes code that has to run somewhere. Without a deployed BE, there's nothing to deploy *to* — story 2 (Postgres) can't connect, story 3 onwards can't ship. This story creates the BE submodule, stands up a minimal app, and makes it reachable on the public internet via the chosen deploy target. It also flips the workspace's lane table so future Inceptions know `backend` is live.

## Acceptance criteria

- [ ] A new submodule exists at `backend/` in the workspace, pointing at a freshly-created GitHub repo under the same owner as the other two submodules.
- [ ] The new repo has its own `CLAUDE.md` describing how to build and run the BE locally (commands, env vars needed, ports).
- [ ] Pushing to the `main` branch of the BE repo triggers an automatic deploy to a hosted environment.
- [ ] Hitting the deployed environment's health path from any computer on the public internet returns a success response with the documented shape (see api-contract).
- [ ] The workspace's `CLAUDE.md` lane table is updated: the `backend` row no longer says "dormant — TBD" and instead points at `backend/<module>/` and `backend/CLAUDE.md`.
- [ ] The deployed environment's URL is recorded in [[../../api-contract#Conventions]] and [[../../open-questions#Q1]] is moved to Resolved.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes (Construction chooses the test shape — likely an HTTP-level test against the local app, plus a one-shot manual verification of the deployed URL).
- The lane's standard build/test commands pass with no regressions. To be documented in the new `backend/CLAUDE.md`.

## Blocked by

- nothing — independently grabbable, but blocks every other story in this Inception.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Construction reads
> the lane's CLAUDE.md (to be authored as part of this story) and may diverge
> from any hint here without re-opening Inception.

- **Stack:** Ktor on JVM. See [[../../decisions#D5]].
- **Deploy target:** Railway. The workspace already has a `railway-deployment` skill that covers Bun/JVM apps + auto-deploy + env vars.
- **Submodule wiring:** mirror the pattern that `weft/` and `undercurrent/` use — a fresh remote repo, then `git submodule add` from the workspace, then commit the workspace's pinned pointer.
- **Watch out for:** the health endpoint must NOT touch the DB (DB is wired in story 2). Keep it static so Railway's healthcheck doesn't depend on DB readiness.
- **Watch out for:** secrets handling — even at this stage there will be at least one env var (the eventual `DATABASE_URL`). Use Railway env vars from day one; don't commit a `.env` placeholder.

## Out of scope for this story

- Postgres / migrations — [[02-postgres-and-migrations]].
- Any auth / account work.
- CI workflow on the new repo — explicitly NOT in v1 per [[../../decisions#D9]]. Do not add a GitHub Actions workflow; Railway's auto-deploy from `main` is the only automated check.
- A custom domain — [[../../open-questions#Q1]] resolves how the URL gets shared; default-guess is Railway's generated URL.
