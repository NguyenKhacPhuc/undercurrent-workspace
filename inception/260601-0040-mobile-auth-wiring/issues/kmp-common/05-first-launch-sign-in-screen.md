---
type: issue
feature: mobile-auth-wiring
lane: kmp-common
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 1
estimate: 90m
blocked-by:
  - "[[01-session-token-store-interface]]"
  - "[[04-be-auth-client]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mobile-auth-wiring
  - status/in-progress
  - wave/1
---

# [kmp-common] First-launch register-or-sign-in screen

**Lane:** kmp-common
**PRD section:** [[PRD#Story 1 — First-launch register or sign-in (blocking)]] + [[PRD#Story 2 — Subsequent launches skip the sign-in screen]]
**API contract section:** [[api-contract#`POST /v1/auth/sign-up`]] + [[api-contract#`POST /v1/auth/sign-in`]]

## Why

This is the user-visible half of the feature. Without it, the BE has no way to receive an account or hand out a session, and the rest of the app (Settings, future authed surfaces) has no signed-in user to render against. Until the user completes this screen, they cannot reach the rest of the app on a fresh install with no stored token.

## Acceptance criteria

- [ ] On app launch with no stored session token in [[01-session-token-store-interface]], the user lands on the sign-in screen instead of the usual home surface.
- [ ] The screen has a mode toggle / picker (UX form per [[../../open-questions#Q2]]) that switches between Sign In (default) and Register; common fields (email, password) keep their values across the toggle; Register adds a displayName field.
- [ ] The Continue action is disabled until all required fields for the active mode are filled and pass the same validation rules the BE will apply (per [[api-contract#Validation rules]]).
- [ ] Tapping Continue in Sign In mode calls `signIn` on the BE client; success persists the returned token via [[01-session-token-store-interface]] and routes the user to the home surface.
- [ ] Tapping Continue in Register mode calls `signUp`; success persists the token and routes the user to the home surface.
- [ ] On a `unauthenticated` result during Sign In, the user sees "Invalid email or password" inline; fields stay filled (email keeps last value, password cleared); nothing is stored.
- [ ] On an `email_already_registered` result during Register, the user sees an inline message AND a one-tap "Switch to Sign In with this email" shortcut that flips the mode and pre-fills the email.
- [ ] On an `invalid_request` result, per-field error messages are shown if the response `details` map is populated; otherwise the BE's `error.message` is shown above the form.
- [ ] On a `rate_limited` result, the user sees "Too many failed attempts. Try again later." inline (no countdown — see [[../../decisions#D5]]).
- [ ] On a network-error result, the user sees "Couldn't reach the server. Check your connection." inline with a Retry action; fields stay filled.
- [ ] On every subsequent app launch with a stored token, the user goes straight to the home surface; the sign-in screen is not shown.
- [ ] If any authed call elsewhere in the app returns `unauthenticated`, the local token is wiped and the user is routed back to this screen (Story 2 AC). On that re-show, the email field is pre-filled from the last known account (per [[../../open-questions#Q3]]) and a brief one-line notice says "You were signed out — please sign in again."

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes. Likely split: a `SignInViewModelStateTest` in commonTest exercising every state transition against fake `SessionTokenStore` + fake `AuthClient`; and a small `SignInViewModelTest` in androidUnitTest with MockK if `coVerify` on collaborator calls is wanted.
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`.

## Blocked by

- [[01-session-token-store-interface]] — needs the interface to persist the issued token.
- [[04-be-auth-client]] — needs the typed client to call sign-up / sign-in.

(Does NOT depend on [[../android/02-android-encrypted-session-token-store]] or [[../ios/03-ios-keychain-session-token-store]] — those are wired in via Koin at app startup; tests use fakes.)

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Likely location:** new `feature/sign-in/` Gradle module (mirrors the other ~17 features). The MVI base class to extend is `MviViewModel` in `:shared`. Routing decision in `ScreenRouter` — see `undercurrent/CLAUDE.md` for the navigation overview.
- **Existing pattern to mirror:** any existing feature module is a fine template (`feature/integrations` is recent + small). The Route + Screen + ViewModel + Module quartet matches `undercurrent/CLAUDE.md`.
- **Watch out for:** the routing splice — sign-in goes BEFORE the existing provider-picker / API-key onboarding per [[../../decisions#D7]]. Confirm where in ScreenRouter the new start-destination check lives.
- **Watch out for:** IME / keyboard behavior on both platforms. Two-to-three text fields stacked; make sure Continue is reachable above the IME on small Android phones, and that password field's secure-text mode is set on both platforms.

## Out of scope for this story

- The Settings account display + Sign Out — [[06-settings-account-and-sign-out]].
- Forgot-password UI — see [[../../out-of-scope]].
- Email verification — see [[../../out-of-scope]].
- "Remember me" toggle — see [[../../out-of-scope]].
