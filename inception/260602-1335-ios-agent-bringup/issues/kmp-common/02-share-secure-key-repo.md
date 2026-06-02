---
type: issue
feature: ios-agent-bringup
lane: kmp-common
status: done
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/13
merged-at: 2026-06-02T08:23:29Z
claimed-by: SteveCastalk
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ios-agent-bringup
  - status/done
  - wave/0
---

# [kmp-common] Share the secure provider-key repository

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 2 — Shared secure-key storage
**API contract section:** n/a (no BE)

## Why

The host stores provider keys behind a provider-keyed repository. On Android that repository delegates to the SDK's secure key vault; on iOS it has its own standalone Keychain copy. Now that the SDK ships a secure key vault on iOS too, the host can use **one shared** provider-key repository that delegates to the SDK on both platforms — and the iOS standalone copy can go.

## Acceptance criteria

- [ ] Saving, reading, checking, and clearing a provider key goes through the SDK's secure key vault on both Android and iOS, via one shared repository.
- [ ] A key saved on iOS is readable after an app restart.
- [ ] The iOS app no longer carries its own separate Keychain key-storage copy.
- [ ] Android key storage behaves exactly as before.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** lift `undercurrent/core/domain/src/androidMain/…/WeftKeyVaultRepository.kt` → `…/commonMain/…`; delete `undercurrent/core/domain/src/iosMain/…/KeychainKeyVaultRepository.kt`.
- **Substrate piece consumed:** the SDK key vault (`KeyVault`, alias-based) — the shared repo maps provider → alias (mirror the Android impl's alias constants).
- **Watch out for:** the SDK's iOS key vault stores under its own service id; values previously written by the host's standalone Keychain copy won't migrate — acceptable (users re-paste keys) but call it out at review.
- **Verify (from `undercurrent/`):** `./gradlew :core:domain:test :core:domain:compileDebugKotlinAndroid :core:domain:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- BE session-token storage (`KeychainSessionTokenStore`) — that's host-specific and stays; not part of this story.
