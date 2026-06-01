---
type: issue
feature: mobile-auth-wiring
lane: kmp-common
status: done
claimed-by: NguyenKhacPhuc
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/1
merged-at: 2026-06-01T02:51:50Z
wave: 0
estimate: 30m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mobile-auth-wiring
  - status/done
  - wave/0
---

# [kmp-common] A session token can be stored, read, and cleared via a KMP interface

**Lane:** kmp-common
**PRD section:** [[PRD#Goals]] (the "stored in the platform's secure store" goal)
**API contract section:** n/a — no BE call

## Why

The sign-in screen, the authed-request layer, and the Sign Out tap all need ONE shared way to ask "is there a token?" / "give me the current token" / "wipe the token." Defining the interface once in commonMain — before the Android and iOS impls land — means the upstream consumers (Story 04's API client, Story 05's ViewModel) can be built and tested against the interface without waiting for either platform's secure-store work to complete.

## Acceptance criteria

Foundation slice — no UI. The contract is observable through the interface's public surface plus a hand-rolled `FakeSessionTokenStore` in `commonTest` that the contract test exercises.

- [ ] An interface exists in commonMain that lets the rest of the app: save a token (string), read the current token (nullable — null means "no session stored"), and clear it.
- [ ] Reading after saving returns the same string within the same instance (in-memory contract for the fake; persistence is each platform impl's concern in stories 02 / 03).
- [ ] Reading before any save has happened returns null.
- [ ] Clearing makes a subsequent read return null.
- [ ] Saving over an existing token replaces it (single-slot — no history).
- [ ] The contract test is written so each platform's impl story can run it against the real impl by parameterizing the spec; the test does NOT assume any platform-specific behavior beyond the four AC bullets above.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes (Construction chooses the test shape).
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :androidApp:compileDebugKotlin`, `./gradlew :composeApp:compileKotlinIosSimulatorArm64`, and the affected module's `test` task.

## Blocked by

- nothing — independently grabbable; blocks 02, 03, 05.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Likely location:** alongside the existing `*Repository` interfaces under `core/domain/src/commonMain/.../`. Naming convention follows project conventions; `SessionTokenStore` is the placeholder.
- **Watch out for:** suspend vs sync — secure-store access on Android is typically synchronous (EncryptedSharedPreferences is blocking), but iOS Keychain access can be async-ish. Suspend functions in the interface give both platforms freedom and don't cost much elsewhere.
- **Watch out for:** the contract test design matters here — Story 02 + 03 will REUSE it against their real impls. Make it parameterizable (e.g. a `tokenStoreContract(factory: () -> SessionTokenStore)` function the platform tests call).

## Out of scope for this story

- Either platform's real impl — those are [[../android/02-android-encrypted-session-token-store]] and [[../ios/03-ios-keychain-session-token-store]].
- The HTTP / Ktor client that consumes the token — [[04-be-auth-client]].
- Any UI surface — [[05-first-launch-sign-in-screen]] and [[../kmp-common/06-settings-account-and-sign-out]].
