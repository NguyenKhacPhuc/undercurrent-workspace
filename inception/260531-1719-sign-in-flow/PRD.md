---
type: prd
feature: sign-in-flow
status: superseded
created: 2026-05-31
superseded: 2026-06-01
tags:
  - inception/prd
  - feature/sign-in-flow
  - status/superseded
---

# PRD: Sign-in flow

> [!warning] **Superseded 2026-06-01.**
> This PRD's `Constraints` block declares "No backend. All persistence is local." That premise was inverted when `backend-bootstrap-auth` shipped end-to-end on 2026-06-01. The replacement Inception is [[../260601-0040-mobile-auth-wiring/PRD]]. See [[decisions#D5 (added 2026-06-01)]] for context.

## One-line intent

Capture a user's display name and email at first launch through a blocking sign-in screen, so Undercurrent knows who the user is locally and has the identity hooks needed when cross-device sync eventually arrives.

## Problem

Undercurrent has no user identity today. Every install is anonymous — the agent cannot address the user by name, and there is no anchor point for the future server-backed sync of conversations / personas / memory across devices. We need to start collecting this information now (cheap, local-only) so the data exists by the time the backend lane wakes up.

The cost of not doing it: when sync ships, we either force a disruptive identity-collection moment onto every existing user, or we accept that their pre-sync data is permanently anonymous and can't be merged with their account.

## Goals

Testable goals. If you can't measure it, rewrite it.

- [ ] On a fresh install, the user cannot reach the chat / home surface without completing sign-in.
- [ ] After sign-in, the captured display name and email survive an app kill + relaunch.
- [ ] An existing installed user (no profile yet, app upgraded into this feature) sees the same blocking sign-in screen on next open.
- [ ] The user can update their display name and email later from Settings, and the new values persist.

## Non-goals

What we are explicitly NOT doing in this feature. Promote anything controversial to `out-of-scope.md`.

- Verifying the email address (no backend → no verification email).
- Authenticating the user against any identity provider (no Sign in with Apple, no Google, no password).
- Syncing the profile to a server, or any cross-device behavior.
- Surfacing the captured name inside the agent's system prompt or greetings.
- Avatars, photos, or any field beyond display name + email.

## User stories

Each story has acceptance criteria. A story without acceptance criteria is not ready to cut into issues.

### Story 1 — First-launch sign-in is blocking

**As a** user opening Undercurrent for the first time (or after upgrading into this release with no profile yet), **I want** to be asked for my name and email before I see the rest of the app, **so that** Undercurrent has my identity from the very first session.

**Acceptance criteria:**
- [ ] When the app launches and no profile is stored, the user lands on a sign-in screen instead of the usual home surface.
- [ ] The sign-in screen has a display-name field and an email field, plus a Continue action.
- [ ] Continue is disabled until both fields are filled.
- [ ] Tapping Continue records the values and takes the user to the usual home surface.
- [ ] On the next app open, the user goes straight to the home surface — the sign-in screen is not shown again.
- [ ] If the app is force-killed mid-sign-in (fields filled but Continue not tapped), nothing is persisted and the sign-in screen reappears next launch.

### Story 2 — Editable profile from Settings

**As a** signed-in user, **I want** to update my display name or email later, **so that** I am not stuck with a typo or an old address forever.

**Acceptance criteria:**
- [ ] Settings shows the user's current display name and email.
- [ ] From Settings, the user can open an editor (inline or a sub-screen — Construction's choice), change either field, and save.
- [ ] Saving persists the new values; cancelling discards changes.
- [ ] After save, Settings shows the new values immediately, and they survive an app restart.

## Success metrics

How we know this feature worked, after launch. At least one.

- **Sign-in completion rate** — % of fresh installs that reach the home surface within 5 minutes of first open. Target: ≥ 95%. Measured via existing telemetry on first home-surface render.
- **Profile-edit usage** — count of profile edits per 100 active users per week. No target — this metric exists to detect whether the editor is discoverable and whether the captured data is wrong often enough to matter.

## Constraints

Deadlines, dependencies, compliance, existing systems we must respect.

- **No backend.** All persistence is local to the device. Use the existing DataStore-Preferences plumbing.
- **KMP-shared.** Both Android and iOS ship this in the same release. No platform-specific impl unless forced by something we discover during Construction.
- **No telemetry of the email itself** off-device. The email is treated as personal data the user typed for their own benefit; we don't send it anywhere right now (and the sign-in screen says so — see [[open-questions#Q4]]).
- **Existing onboarding.** This release is the first time we add a blocking pre-home gate. Construction must check how it interleaves with the existing "pick a provider / paste API key" flow ([[open-questions#Q6]]).

## Links

- API contract: [[api-contract]]
- Open questions: [[open-questions]]
- Decisions: [[decisions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]
- Issues: `./issues/kmp-common/`
