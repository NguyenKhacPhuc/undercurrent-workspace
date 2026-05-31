---
type: prd
feature: mobile-auth-wiring
status: draft
created: 2026-06-01
tags:
  - inception/prd
  - feature/mobile-auth-wiring
  - status/draft
---

# PRD: Mobile auth wiring

> [!info] **Status:** Draft / awaiting mob review · **Driver:** Phuc · **Last updated:** 2026-06-01
> Replaces the superseded [[../260531-1719-sign-in-flow/PRD]]. See [[decisions#D1]] for why this is a new Inception rather than a revision of the old one.

## One-line intent

Wire the Android + iOS app to the shipped backend's auth endpoints so users register or sign in with email + password, the 30-day bearer token is securely stored on-device, and any authed request the app makes carries that token.

## Problem

The `backend-bootstrap-auth` Inception shipped end-to-end on 2026-06-01 — `POST /v1/auth/sign-up`, `POST /v1/auth/sign-in`, `GET /v1/me`, `POST /v1/auth/sign-out` all work in production. Nothing on the mobile side calls them yet, so the BE is currently invisible to users.

This Inception puts a face on the BE: a first-launch sign-in screen that lets a user register or sign in, plus the plumbing to store the resulting session token securely and surface the current account in Settings. After this lands, the app has a real user identity it can attach future user-scoped features to (sync, multi-device, etc.).

The cost of not doing it: every install stays anonymous, the BE we just paid for sits idle, and the next user-scoped feature has to wait or duplicate this work.

## Goals

Testable goals. If you can't measure it, rewrite it.

- [ ] On a fresh install with no stored session token, the user cannot reach the chat / home surface until they complete register-or-sign-in against the BE.
- [ ] After a successful register or sign-in, the issued bearer token is stored in the platform's secure store (Keychain on iOS, EncryptedSharedPreferences on Android) — NOT in plain `DataStore-Preferences`.
- [ ] On every subsequent app launch with a valid token, the user goes straight to the home surface without seeing the sign-in screen again.
- [ ] Settings shows the current account's display name + email by fetching `GET /v1/me` on entry (BE is source of truth — no persisted mirror).
- [ ] Settings has a "Sign Out" affordance that calls `POST /v1/auth/sign-out`, wipes the local token, and returns the user to the sign-in screen.
- [ ] When the BE returns 401 to any authed request after launch, the app wipes the local token and routes back to the sign-in screen (handles server-side revoke + 30-day expiry the same way).

## Non-goals

What we are explicitly NOT doing in this feature. Promote anything controversial to `out-of-scope.md`.

- Editing the display name or email after sign-in. The BE does not yet expose `PATCH /v1/me`; until it does, Settings shows the account read-only. ([[../260531-1733-backend-bootstrap-auth/out-of-scope]] tracks the BE side.)
- Forgot-password / password reset. The BE does not yet expose this endpoint; the mobile UI does NOT pretend it does (no fake link).
- Email verification on the client. The BE accepts unverified emails (deferred); we trust whatever the server returns.
- Sign in with Apple / Google. Out of scope; future Inception.
- Syncing conversations, personas, mini-apps, memory, or any on-device data the user has built up. Future Inception per data type.
- Offline mode / queued auth attempts. Sign-in requires a working network; we surface that clearly when it isn't.
- Migration of the (never-shipped) local profile from the superseded Inception. No mobile code from `260531-1719-sign-in-flow/` ever landed in production, so there is no on-device profile to migrate.

## User stories

Each story has acceptance criteria.

### Story 1 — First-launch register or sign-in (blocking)

**As a** user opening Undercurrent for the first time on a device with no stored session, **I want** to either create a new account or sign in to my existing one with email + password, **so that** the BE knows who I am and the app has a session token to use on every subsequent authed call.

**Acceptance criteria:**

- [ ] On app launch with no stored session token, the user lands on a sign-in screen instead of the usual home surface.
- [ ] The screen supports both register (new user) and sign-in (returning user) modes — UX form per [[open-questions#Q2]].
- [ ] Register asks for display name, email, password; the Continue action is disabled until all three are filled and pass the same validation the BE will apply (see [[open-questions#Q1]]).
- [ ] Sign-in asks for email + password; the Continue action is disabled until both are filled.
- [ ] Tapping Continue submits to the BE (`/v1/auth/sign-up` for register, `/v1/auth/sign-in` for sign-in), stores the returned session token in the platform's secure store, and routes the user to the home surface.
- [ ] On a BE 400 (validation error), the user sees the BE's error message inline; fields stay filled; nothing is stored.
- [ ] On a BE 409 (`email_already_registered`, register only), the user sees an inline message and a one-tap shortcut to switch to sign-in mode with the email pre-filled.
- [ ] On a BE 401 (`unauthenticated`, sign-in only), the user sees the BE's "Invalid email or password" message inline.
- [ ] On a BE 429 (`rate_limited`, sign-in only), the user sees "Too many failed attempts. Try again later." inline. (No countdown timer in v1.)
- [ ] On a network error (timeout, unreachable, no internet), the user sees "Couldn't reach the server. Check your connection." inline with a Retry action; fields stay filled.
- [ ] If the app is force-killed mid-form (fields filled but Continue not tapped), nothing is stored and the sign-in screen reappears next launch with empty fields.

### Story 2 — Subsequent launches skip the sign-in screen

**As a** signed-in user, **I want** the app to remember I'm signed in across launches, **so that** I don't see the sign-in screen on every cold start.

**Acceptance criteria:**

- [ ] On app launch with a stored session token, the user lands on the home surface (sign-in screen is NOT shown).
- [ ] If any authed request the app makes returns 401, the local token is wiped and the user is routed back to the sign-in screen. (Handles server-side revoke + natural 30-day expiry the same way.)

### Story 3 — Settings shows the current account, plus Sign Out

**As a** signed-in user, **I want** Settings to show me which account I'm signed in to and let me sign out, **so that** I can confirm my identity and revoke the device's session when I want.

**Acceptance criteria:**

- [ ] Opening Settings triggers a `GET /v1/me` and shows the returned display name + email in a read-only "Account" section.
- [ ] While `GET /v1/me` is in flight, the Account section shows a loading placeholder.
- [ ] If `GET /v1/me` fails with a network error, the Account section shows "Couldn't load account" with a Retry; failure does NOT log the user out.
- [ ] If `GET /v1/me` fails with 401, the local token is wiped and the user is routed back to the sign-in screen (per Story 2 AC).
- [ ] Settings has a "Sign Out" affordance below the Account section.
- [ ] Tapping Sign Out calls `POST /v1/auth/sign-out` (best-effort — succeeds on 204 OR on network error, see [[decisions#D5]]), wipes the local token, and returns the user to the sign-in screen.

## Success metrics

How we know this feature worked, after launch. At least one.

- **Sign-in completion rate** — % of fresh installs that reach the home surface within 5 minutes of first open. Target: ≥ 95% over the first month. Established via existing first-home-surface telemetry.
- **Authed-401-recoveries-per-week** — count of 401-induced sign-out-and-return-to-sign-in events. No target; metric exists to spot a token-expiry hot spot once we have real users (e.g. if it's > 1 per user per week, we have a session lifecycle bug).
- **Sign-out usage** — count of Sign Out taps per active user per week. No target; ensures the affordance is discoverable.

## Constraints

Deadlines, dependencies, compliance, existing systems we must respect.

- **BE shape is fixed.** The shipped BE's request/response shapes (`AuthResponse`, `ErrorEnvelope`, 401/400/409/429 mapping) are the contract — see [[api-contract]] for the canonical reference. Mobile mirrors the BE exactly; any divergence is a bug to fix on the mobile side.
- **Secure token storage required.** The 30-day bearer is a high-value secret; storing it in plain `DataStore-Preferences` is unacceptable. Per-platform secure store is the bar. Re-opens [[../260531-1719-sign-in-flow/decisions#D4]] (100% commonMain stance) — see [[decisions#D2]].
- **Sign-in goes before provider-picker / API-key onboarding.** Resolves [[../260531-1719-sign-in-flow/open-questions#Q6]] — identity first, then config. Sign-in is its own screen; no merging with the provider step. See [[decisions#D7]].
- **No new BE work in this Inception.** All endpoints already exist. If we discover we need PATCH /v1/me or anything else, we surface it back into a new BE Inception, not bolt it on here.

## Links

- API contract: [[api-contract]]
- Open questions: [[open-questions]]
- Decisions: [[decisions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[../../CONTEXT]]
- Backend Inception: [[../260531-1733-backend-bootstrap-auth/PRD]]
- Superseded predecessor: [[../260531-1719-sign-in-flow/PRD]]
- Issues: `./issues/`
