---
type: issue
feature: mobile-auth-wiring
lane: android
status: done
claimed-by: NguyenKhacPhuc
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/2
merged-at: 2026-06-01T07:27:09Z
wave: 1
estimate: 45m
blocked-by:
  - "[[../kmp-common/01-session-token-store-interface]]"
tags:
  - inception/issue
  - lane/android
  - feature/mobile-auth-wiring
  - status/done
  - wave/1
---

# [android] Session token is stored in EncryptedSharedPreferences

**Lane:** android
**PRD section:** [[PRD#Goals]] (Keychain on iOS / EncryptedSharedPreferences on Android)
**API contract section:** n/a — no BE call

## Why

Per [[../../decisions#D2]], the 30-day bearer cannot live in plain DataStore-Preferences on Android — it's a high-value secret. EncryptedSharedPreferences ties the encryption key to the device's keystore alias and gives us the standard mobile pattern without an in-house crypto layer.

## Acceptance criteria

- [ ] An `androidMain` implementation of [[../kmp-common/01-session-token-store-interface]] backs all four operations (save / read / clear / overwrite) by `EncryptedSharedPreferences`.
- [ ] The contract test from story 01 passes when run against this real impl in `androidUnitTest` (or `androidInstrumentedTest` if `EncryptedSharedPreferences` requires a real Android context — Construction picks).
- [ ] A token written through this impl survives an app process kill (verified end-to-end by an instrumentation-style smoke OR by manually killing the test process and re-reading).
- [ ] Uninstall + reinstall clears the token cleanly (no leftover encrypted file the new install can't read).
- [ ] The Koin binding for `SessionTokenStore` resolves to this impl in the Android `Application` module.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :<module>:testDebugUnitTest :androidApp:compileDebugKotlin`.

## Blocked by

- [[../kmp-common/01-session-token-store-interface]] — needs the interface + the reusable contract test.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Likely dep:** `androidx.security:security-crypto`. This is a NEW dep on the Android side — log it in the workspace decisions when adding (per construct skill rule).
- **Likely location:** `core/domain/src/androidMain/.../`. The existing `RepositoryAndroidModule.kt` is the natural Koin wire-up point — confirm against `undercurrent/CLAUDE.md`.
- **Watch out for:** `EncryptedSharedPreferences` requires a `MasterKey` set up with a `KeyGenParameterSpec`. The defaults are fine (AES-256-GCM); just make sure you use the AndroidX-recommended invocation, not the older deprecated API.
- **Watch out for:** test surface — pure unit tests can't fully exercise `EncryptedSharedPreferences` because it depends on AndroidX Security's keystore. Either run the contract test in `androidInstrumentedTest` or wrap the encrypted store in a thin abstraction so the unit test exercises everything ABOVE the storage call.

## Out of scope for this story

- iOS impl — [[../ios/03-ios-keychain-session-token-store]] (parallel).
- The HTTP client consuming the token — [[04-be-auth-client]].
- The sign-in screen UI — [[05-first-launch-sign-in-screen]].
