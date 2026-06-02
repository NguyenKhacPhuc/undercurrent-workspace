---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 20m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/0
---

# [Substrate] The agent can read and write the clipboard on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

Copy/paste is a basic agent action (hand the user a result, pick up what they copied). Works on Android; unimplemented on iOS.

## Acceptance criteria

- [ ] The agent can place text on the iOS clipboard and the user can paste it elsewhere.
- [ ] The agent can read text the user has copied on iOS.
- [ ] Reading an empty clipboard returns "nothing copied" rather than failing.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/clipboard/IosClipboard.kt` (stub).
- **Existing pattern to mirror:** the Android clipboard impl in `:os-bridge`.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Clipboard-change watching beyond what the Android impl exposes.
- Wiring into the one-call setup — [[14-ios-one-call-setup]].
