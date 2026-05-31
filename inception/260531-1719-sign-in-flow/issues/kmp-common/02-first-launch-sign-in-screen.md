---
type: issue
feature: sign-in-flow
lane: kmp-common
status: superseded
superseded: 2026-06-01
wave: 1
estimate: 90m
blocked-by:
  - "[[01-user-profile-store]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/sign-in-flow
  - status/superseded
  - wave/1
---

> [!warning] **Superseded 2026-06-01 by `260601-0040-mobile-auth-wiring/`.** Replacement splits this into register (new user → BE sign-up) and sign-in (returning user → BE sign-in) modes, adds a password field, and adds secure token storage (Keychain/EncryptedSharedPreferences).

# [kmp-common] First-launch blocking sign-in screen

**Lane:** kmp-common
**PRD section:** [[PRD#Story 1 — First-launch sign-in is blocking]]
**API contract section:** n/a — no backend

## Why

This is the visible half of the feature. Without it, the app on a fresh install (or an upgraded install with no profile yet) has no way to ask the user for their name and email, and the work in [[01-user-profile-store]] never gets exercised. Until the user completes this screen, they cannot reach the rest of the app.

## Acceptance criteria

- [ ] On app launch, if no user profile is stored yet, the user lands on the sign-in screen instead of the usual home surface.
- [ ] The sign-in screen has a display-name field, an email field, and a Continue action.
- [ ] Continue is disabled until both fields contain something the user typed (after trim).
- [ ] If a user types a clearly malformed email (no `@`, or no `.` after the `@`, or contains spaces), they see an inline error and Continue stays disabled.
- [ ] If the display name is just whitespace, they see an inline error and Continue stays disabled.
- [ ] Tapping Continue records the values and takes the user to the usual home surface.
- [ ] On every subsequent app launch, the user goes straight to the home surface — the sign-in screen is not shown again.
- [ ] If the user force-kills the app while filling in fields (before tapping Continue), nothing is persisted and the sign-in screen reappears on next launch.
- [ ] On an install that was upgraded into this release with no previous profile, the same blocking sign-in screen is shown on next open — identical experience to a fresh install. (See [[../../decisions#D3]].)
- [ ] A short line of helper text under the email field tells the user where their email goes (driver guess: "Stored only on this device. We don't send your email anywhere yet." — final copy depends on [[../../open-questions#Q4]]).

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes (Construction chooses the test shape).
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :androidApp:compileDebugKotlin`, `./gradlew :composeApp:compileKotlinIosSimulatorArm64`, and the affected module's `test` task.

## Blocked by

- [[01-user-profile-store]] — the screen has nothing to write to until the store exists.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Construction reads
> the lane's CLAUDE.md + opens existing similar files and may diverge from
> any hint here without re-opening Inception.

- **Likely files affected:** new `feature/sign-in/` Gradle module (mirrors the other ~17 features). Routing decision lives wherever `ScreenRouter` selects the start destination — see `undercurrent/CLAUDE.md` for the navigation overview. The MVI base class to extend is `MviViewModel` in `:shared`.
- **Existing pattern to mirror:** any existing feature module is a fine template (`feature/integrations` is recent and small). The Route + Screen + ViewModel + Module quartet matches the conventions in `undercurrent/CLAUDE.md`.
- **Watch out for:** ordering vs the existing provider-picker / API-key onboarding. Driver guess: sign-in first, provider step second. See [[../../open-questions#Q6]] — confirm with whoever owns onboarding today.
- **Watch out for:** IME / keyboard behavior on both platforms. The screen has two text fields right next to each other — make sure the email field's `KeyboardType` is set, and that Continue is reachable above the IME on small Android phones.

## Out of scope for this story

- The Settings editor for changing the values later — that's [[03-edit-profile-from-settings]].
- Surfacing the captured display name elsewhere in the app (agent greetings, system prompt, home-surface header). See [[../../open-questions#Q3]] and the PRD non-goals.
- A "Reset profile" or sign-out option. See [[../../open-questions#Q7]].
