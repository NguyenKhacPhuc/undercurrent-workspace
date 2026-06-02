---
type: issue
feature: weft-ios-parity
lane: substrate
status: done
merged-pr: https://github.com/NguyenKhacPhuc/android-harness/pull/7
merged-at: 2026-06-02T06:00:34Z
claimed-by: SteveCastalk
wave: 0
estimate: 30m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/done
  - wave/0
---

# [Substrate] The agent can trigger haptic feedback on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

Haptic taps confirm actions and signal success/failure without sound. Works on Android; unimplemented on iOS.

## Acceptance criteria

- [ ] When the agent triggers a haptic effect on iOS, the device produces the corresponding feedback.
- [ ] Each haptic effect the SDK exposes maps to a distinct, sensible iOS feedback (e.g. light tap vs. success vs. warning).
- [ ] On a device without haptic support, the call completes quietly rather than failing.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/haptics/IosHaptics.kt` (stub).
- **Existing pattern to mirror:** the Android haptics impl + the shared haptic-effect enum in `:contracts`.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- New haptic effects not already in the shared enum.
