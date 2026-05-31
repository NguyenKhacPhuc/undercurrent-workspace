---
type: prd
feature: backend-bootstrap-auth
status: draft
created: 2026-05-31
tags:
  - inception/prd
  - feature/backend-bootstrap-auth
  - status/draft
---

# PRD: Backend bootstrap + email/password auth

> [!info] **Status:** Draft / awaiting mob review · **Driver:** Phuc · **Last updated:** 2026-05-31
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

Stand up Undercurrent's first ever backend — a deployed, internet-reachable service that can register new user accounts with email + password, sign them in, identify the current user, and sign them out.

## Problem

Undercurrent has been client-only since day one. Every install is anonymous, and the [[../260531-1719-sign-in-flow/PRD|sign-in flow Inception]] that just landed only stores the user's typed identity locally. Without a server, there is no way to:

- recognize the same person across two devices,
- recover an account after a reinstall, or
- attach anything user-scoped (memory sync, conversation sync, shared mini-apps) to a stable identity.

This Inception covers **only** the foundation: the BE exists, it is deployed, it owns the account record, and a person can sign up / sign in / be identified / sign out. Cross-device sync of conversations / personas / memory is a future Inception that builds on this one.

The cost of not doing it: every adjacent feature (sync, recovery, shared anything) is permanently blocked until a BE exists. The cost of doing it cheaply now: the team has to learn one new submodule and one new deploy target.

## Goals

Testable goals. If you can't measure it, rewrite it.

- [ ] A live BE is reachable on the public internet at a stable URL and returns success from a health endpoint.
- [ ] A new user can register with display name + email + password and immediately make authenticated requests.
- [ ] An existing user can sign in with email + password and immediately make authenticated requests.
- [ ] An authenticated request can be exchanged for the current user's display name + email + stable id.
- [ ] An authenticated user can sign out, after which their previous credentials no longer authenticate any request.
- [ ] The BE shrugs off password-grinding: after enough failed sign-in attempts against the same email, further attempts are throttled for a window.

## Non-goals

What we are explicitly NOT doing in this feature. Promote anything controversial to `out-of-scope.md`.

- Mobile-client integration of these endpoints. (Driver chose BE-only this Inception — see [[decisions#D4]]. A follow-up Inception will wire the existing sign-in screen to call the new endpoints.)
- Forgot-password / password reset.
- Email verification on sign-up.
- Change password while signed in.
- Any sync of conversations, personas, mini-apps, memory.
- Any third-party identity (Sign in with Apple, Google, etc.).
- Multi-account on a single install.

## User stories

Each story has acceptance criteria.

> [!note] The "user" for this Inception is split. The **API consumer** is the mobile client (and Postman, and the BE team's curl). The **end user** is the human behind the device. Stories below speak to whichever is the right level of observation — most are API-consumer level because this is BE-only.

### Story 1 — There is a deployed backend

**As the** BE team, **I want** a live Ktor app on Railway with a public URL and a health endpoint, **so that** every other BE story has a place to land and we have something to point at.

**Acceptance criteria:**
- [ ] A `backend/` submodule exists in the workspace, pinned at a known commit.
- [ ] The `main` branch of that submodule auto-deploys to a Railway environment on every push.
- [ ] Hitting the deployed URL's health path returns success from anywhere on the public internet.
- [ ] The workspace's `CLAUDE.md` lane table is updated so `backend` is no longer marked dormant.

### Story 2 — A persistent store is wired

**As the** BE team, **I want** a Postgres database connected to the running app with migrations applied on startup, **so that** the upcoming Account and Session stories have a place to keep data across restarts.

**Acceptance criteria:**
- [ ] The deployed app boots successfully against a Railway-managed Postgres.
- [ ] An empty no-op migration runs on startup and is recorded as having run.
- [ ] Restarting the app does not re-run an already-applied migration.

### Story 3 — Sign-up creates an account

**As a** new user, **I want** to register with my display name, email, and password, **so that** the BE knows who I am and I am immediately signed in.

**Acceptance criteria:**
- [ ] A sign-up request with a valid display name, valid email, and valid password creates an account and responds with the new account's stable id, display name, email, plus an opaque session token the caller can use for authenticated requests.
- [ ] The password is stored as a salted hash, never as plain text.
- [ ] Signing up with an email that already has an account fails with a clear conflict error and does not create a duplicate.
- [ ] Signing up with a malformed email, a too-short password, or an empty display name fails with a clear validation error and does not create an account.
- [ ] The session token issued from sign-up is immediately usable on subsequent authenticated requests.

### Story 4 — Sign-in authenticates an existing account

**As a** returning user, **I want** to sign in with email and password, **so that** I can resume making authenticated requests.

**Acceptance criteria:**
- [ ] A sign-in request with a known email and correct password responds with the account's id, display name, email, plus an opaque session token.
- [ ] A sign-in request with the wrong password OR an unknown email fails with the same single error message, so callers cannot probe whether an email is registered.
- [ ] The session token issued from sign-in is immediately usable on subsequent authenticated requests.
- [ ] A successful sign-in does not invalidate any previously-issued session for the same account. (A user signed in on two devices stays signed in on both.)

### Story 5 — The BE can identify the signed-in user

**As an** authenticated client, **I want** to exchange my session token for the current user's id, display name, and email, **so that** mobile UI can show "you are signed in as X".

**Acceptance criteria:**
- [ ] A request with a valid session token responds with the account's id, display name, and email.
- [ ] A request with no session token, an unknown session token, or a revoked one fails with an unauthenticated error.

### Story 6 — Sign-out invalidates a session

**As an** authenticated client, **I want** to sign out, **so that** my session token can no longer be used to act as me.

**Acceptance criteria:**
- [ ] A sign-out request with a valid session token responds with success and invalidates that exact token.
- [ ] After sign-out, a subsequent request reusing the same token fails with an unauthenticated error.
- [ ] Sign-out does not invalidate the account's *other* sessions (only the token presented).
- [ ] Sign-out with a missing or already-invalid token still responds success-shaped (no information leak; idempotent).

### Story 7 — Sign-in throttles brute-force attempts

**As the** BE owner, **I want** repeated failed sign-in attempts on the same email to be throttled, **so that** an attacker cannot grind passwords by spraying the live endpoint.

**Acceptance criteria:**
- [ ] After enough failed sign-in attempts against the same email within a short window, further attempts on that email respond with a throttled error instead of running the password check — regardless of whether the next attempt would have been correct.
- [ ] Throttling clears automatically after a quiet window with no failed attempts.
- [ ] Throttling on one email does not affect sign-in on a different email.
- [ ] Exact thresholds for "enough" and "the window" are spec'd in [[open-questions#Q3]]; PRD goal is only that some such throttling exists and is observable.

## Success metrics

How we know this feature worked, after launch. At least one.

- **Service availability** — uptime of the `/health` endpoint, measured by UptimeRobot pinging every 5 minutes (per [[decisions#D8]]). Target ≥ 99% in the first month. (Establishes that the BE is actually live and we operate it, not just deployed once.)
- **Sign-up → first authenticated request latency** — wall-clock from sign-up response to the next authenticated request from the same session token, measured server-side. Target P95 ≤ 1s end-to-end. Catches an issued-but-not-yet-usable session regression.
- **Rate-limit trip count** — # of sign-in requests blocked by rate-limiting per week. No target — this metric exists to show the throttle is non-trivially exercised (or, if it stays at zero, to question whether it's wired).

## Constraints

Deadlines, dependencies, compliance, existing systems we must respect.

- **First-ever BE.** This release adds the `backend` lane to a workspace that's been client-only. Workspace `CLAUDE.md` lane table needs updating; `backend/CLAUDE.md` is a new artifact. The forked Inception skill's BE default ("skip api-contract — no BE") needs to flip going forward.
- **Existing client.** The sign-in screen Inception ([[../260531-1719-sign-in-flow/PRD]]) does not call this BE. Its [[../260531-1719-sign-in-flow/decisions#D2]] ("no future-proofing for sync") and [[../260531-1719-sign-in-flow/open-questions#Q5]] need re-opening once this BE Inception lands. Tracked here as [[open-questions#Q5]].
- **No secrets in repo.** The Postgres URL, session-signing key (if any), and any other config must come from Railway environment variables, never committed.

## Links

- API contract: [[api-contract]]
- Open questions: [[open-questions]]
- Decisions: [[decisions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[../../CONTEXT]]
- Sibling Inception (client side): [[../260531-1719-sign-in-flow/PRD]]
- Issues: `./issues/backend/`
