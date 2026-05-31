---
type: issue
feature: mobile-auth-wiring
lane: ios
status: ready
wave: 1
estimate: 45m
blocked-by:
  - "[[../kmp-common/01-session-token-store-interface]]"
tags:
  - inception/issue
  - lane/ios
  - feature/mobile-auth-wiring
  - status/ready
  - wave/1
---

# [ios] Session token is stored in the iOS Keychain

**Lane:** ios
**PRD section:** [[PRD#Goals]] (Keychain on iOS / EncryptedSharedPreferences on Android)
**API contract section:** n/a — no BE call

## Why

Per [[../../decisions#D2]], the 30-day bearer needs a per-platform secure store on iOS. Keychain Services is the standard answer and what existing iOS-side bits in this codebase (e.g. `KeychainKeyVaultRepository` for LLM keys) already use.

## Acceptance criteria

- [ ] An `iosMain` implementation of [[../kmp-common/01-session-token-store-interface]] backs all four operations (save / read / clear / overwrite) by the iOS Keychain via `Security.framework`.
- [ ] The contract test from story 01 passes when run against this real impl (in `iosTest` if the harness supports it, or via a quick manual smoke if not).
- [ ] A token written by this impl persists across an app cold-launch.
- [ ] The Koin binding for `SessionTokenStore` resolves to this impl in the iOS Koin module (`composeApp/src/iosMain/.../IosKoinModule.kt` — see `undercurrent/CLAUDE.md`).

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :composeApp:compileKotlinIosSimulatorArm64`.

## Blocked by

- [[../kmp-common/01-session-token-store-interface]] — needs the interface + the reusable contract test.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Existing pattern to mirror:** `core/domain/src/iosMain/.../KeychainKeyVaultRepository.kt` already does Keychain via `platform.Security` interop. Copy its query-dictionary + status-check shape; the auth-token entry differs only in `kSecAttrService` / `kSecAttrAccount` naming.
- **Watch out for:** Keychain entries survive app uninstall on iOS (known platform behavior). Document in the PR's notes; the Sign Out flow ([[06-settings-account-and-sign-out]]) is the user's clean path to remove the token.
- **Watch out for:** the contract test from story 01 should run as-is against this impl — if it doesn't, the issue is the interface, not the impl. Don't paper over a mismatch by editing the contract here.

## Out of scope for this story

- Android impl — [[../android/02-android-encrypted-session-token-store]] (parallel).
- HTTP client — [[04-be-auth-client]].
- UI — [[05-first-launch-sign-in-screen]].
