---
type: issue
feature: weft-ios-parity
lane: substrate
status: done
merged-pr: https://github.com/NguyenKhacPhuc/android-harness/pull/12
merged-at: 2026-06-02T06:00:51Z
claimed-by: SteveCastalk
wave: 1
estimate: 75m
blocked-by:
  - "[[02-ios-credential-vault]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/done
  - wave/1
---

# [Substrate] Signing in to an external provider works turnkey on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 5 — Turnkey sign-in on iOS
**API contract section:** n/a (consumes external provider OAuth we don't author)

## Why

The agent connects to external services (today: Linear) via a provider sign-in flow. The shared protocol/crypto already work on iOS, but there's no iOS launcher for the sign-in web flow or the return-to-app step — so each iOS host wires its own. Provide the launcher in the SDK so sign-in is turnkey, matching Android.

## Acceptance criteria

- [ ] On iOS, starting a connection opens the provider's sign-in page in the system's secure sign-in view.
- [ ] On successful sign-in, the flow returns to the app and the connection is established.
- [ ] If the user cancels the sign-in view, the app returns cleanly with no connection made and the agent is told it was cancelled.
- [ ] The resulting session is stored securely and survives an app restart (the connection stays active).

## Blocked by

- [[02-ios-credential-vault]] — the issued session must be stored securely.

## Hints (non-binding)

- **Likely files affected:** `weft/oauth/src/iosMain/…` — the iOS sign-in launcher + return-to-app handling. The shared protocol/crypto already exist (`weft/oauth/src/commonMain`, `…/iosMain/…Crypto.ios.kt`).
- **Existing pattern to mirror:** the Android launcher flow in `:oauth`. iOS uses the system secure web-auth session; keep the return-to-app contract identical so host code is platform-agnostic.
- **Watch out for:** the return-to-app step needs the host's URL scheme; surface what the host must register rather than hard-coding it. Token storage shape must match [[02-ios-credential-vault]].
- **Verify (from `weft/`):** `./gradlew :oauth:compileKotlinIosSimulatorArm64` · `./gradlew :oauth:compileDebugKotlinAndroid` · `./gradlew :oauth:test` · `./gradlew detekt`

## Out of scope for this story

- New providers beyond what already exists — this is the iOS launcher only.
