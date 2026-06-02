---
type: issue
feature: ios-agent-bringup
lane: ios
status: in-progress
claimed-by: SteveCastalk
wave: 3
estimate: 75m
blocked-by:
  - "[[06-ios-adopt-shared-layer]]"
tags:
  - inception/issue
  - lane/ios
  - feature/ios-agent-bringup
  - status/ready
  - wave/3
---

# [iOS] The iOS app orchestrates the agent lifecycle

**Lane:** iOS (`undercurrent/`)
**PRD section:** [[PRD]] → Story 4 — App lifecycle parity
**API contract section:** n/a (no BE)

## Why

A live chat repo isn't enough — the app has to manage the agent's lifecycle the way Android does: rebuild the agent when the user switches provider or agent, surface permission-denied failures as a dialog, and route the launch flow correctly. This brings the iOS app to behavioral parity.

## Acceptance criteria

- [ ] Switching provider or agent rebuilds the agent; the next reply uses the new selection.
- [ ] Saving a new key resumes the agent.
- [ ] A tool that fails on a missing permission surfaces a dialog with an "Open Settings" action (not an inline error bubble).
- [ ] The launch flow lands the user correctly: sign-in → onboarding → key-paste (if no key) → chat.
- [ ] Navigation stays honest — no stale agent when moving between screens.

## Blocked by

- [[06-ios-adopt-shared-layer]] — needs the agent live first.

## Hints (non-binding)

- **Likely files affected:** `undercurrent/composeApp/src/iosMain/…/IosAppViewModel.kt` — bring it to parity with the shared `WeftAppViewModel` (agent-session integration, provider switching, permission-dialog detection, nav channel). Consider sharing the orchestrator if it's not already.
- **Watch out for:** the permission-denied detection shape (the app reads a tool-fail signal and shows a dialog) — mirror the Android app's handling. Some app config may need the iOS usage-description strings (mic/speech are already present per the substrate's host requirement).
- **Verify (from `undercurrent/`):** `./gradlew :composeApp:compileKotlinIosSimulatorArm64` + iOS app build; manual flow (switch provider, change key, trigger a permission failure).

## Out of scope for this story

- OAuth integrations ([[08-ios-integrations-signin]]).
- Voice / model-catalog / key-validation — deferred ([[out-of-scope]]).
