---
type: issue
feature: ios-agent-bringup
lane: ios
status: done
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/11
merged-at: 2026-06-02T08:11:31Z
claimed-by: SteveCastalk
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/ios
  - feature/ios-agent-bringup
  - status/done
  - wave/0
---

# [iOS] The iOS app stands up the real SDK at launch

**Lane:** iOS (`undercurrent/`)
**PRD section:** [[PRD]] → Story 3 — The agent answers on iOS (prerequisite)
**API contract section:** n/a (no BE)

## Why

The iOS app never constructs the real Weft runtime — it wires stubs. With the substrate's turnkey one-call setup available on iOS, the app can build a real runtime at startup and make it injectable, the same way the Android app does. Everything else in this feature depends on this.

## Acceptance criteria

- [ ] At launch, the iOS app constructs a real SDK runtime via the substrate's single setup call.
- [ ] The runtime is injectable wherever the app needs it (history, agent, etc.).
- [ ] The same agent declarations the Android app registers (e.g. default + writer) are present on iOS.
- [ ] The iOS app still launches cleanly.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `undercurrent/composeApp/src/iosMain/…/IosKoinModule.kt` (add the runtime binding), mirroring `undercurrent/androidApp/…/di/AppModule.kt`'s runtime creation.
- **Substrate piece consumed:** the turnkey iOS `WeftRuntime.create(WeftPlatform(), uiBridge = …, appPromptPreamble = …)` — defaults the OS capabilities; no stand-ins needed.
- **Watch out for:** the app preamble + agent declarations should match Android (pull from the shared `core/AppPreamble`). Branch from `undercurrent` `origin/main` ([[decisions]] D4).
- **Verify (from `undercurrent/`):** `./gradlew :composeApp:compileKotlinIosSimulatorArm64` + the iOS app build (`undercurrent/CLAUDE.md`).

## Out of scope for this story

- Replacing the repository stubs that consume the runtime (that's [[06-ios-adopt-shared-layer]]).
