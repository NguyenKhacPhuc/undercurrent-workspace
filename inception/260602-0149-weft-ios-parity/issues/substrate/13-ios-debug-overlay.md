---
type: issue
feature: weft-ios-parity
lane: substrate
status: done
merged-pr: https://github.com/NguyenKhacPhuc/android-harness/pull/17
merged-at: 2026-06-02T06:18:51Z
claimed-by: SteveCastalk
wave: 1
estimate: 120m
blocked-by:
  - "[[01-ios-shared-sdk-composition]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/done
  - wave/1
---

# [Substrate] The debug overlay is available on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 6 — Debug overlay on iOS
**API contract section:** n/a (no BE)

## Why

Android developers get a debug overlay that shows what the agent is doing this session — invaluable while building. On iOS it's entirely unavailable because the overlay reads the SDK's composition, which only became shareable once [[01-ios-shared-sdk-composition]] landed. Bring the overlay to iOS.

## Acceptance criteria

- [ ] On iOS, a developer can open the debug overlay.
- [ ] The overlay shows the agent activity for the current session (the turns and what the agent did).
- [ ] Opening and closing the overlay does not disrupt the running agent.

## Blocked by

- [[01-ios-shared-sdk-composition]] — the overlay reads the now-shared composition.

## Hints (non-binding)

- **Likely files affected:** `weft/devtools/src/iosMain/…` (today the module has no iOS sources).
- **Existing pattern to mirror:** the Android debug panel in `:devtools` — match what it surfaces. Some Android-only UI helpers (clipboard, date formatting) need shared/iOS equivalents.
- **Watch out for:** scope — see [[open-questions]] Q2. Aim for "good enough to inspect a session", not full feature-for-feature parity with the Android panel.
- **Verify (from `weft/`):** `./gradlew :devtools:compileKotlinIosSimulatorArm64` · `./gradlew :devtools:compileDebugKotlinAndroid` · `./gradlew :devtools:test` · `./gradlew detekt`

## Out of scope for this story

- Any debug-panel feature not already present on Android.
