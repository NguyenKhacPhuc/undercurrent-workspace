---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 30m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/0
---

# [Substrate] The agent can keep the screen awake and adjust brightness on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

Some agent flows need the screen to stay on (reading aloud, a timed activity) or dimmed/brightened. Works on Android; unimplemented on iOS.

## Acceptance criteria

- [ ] When the agent asks to keep the screen awake on iOS, the screen does not auto-dim until released.
- [ ] Releasing the keep-awake request restores normal auto-dim behaviour.
- [ ] The agent can set screen brightness on iOS and the change is visible.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/power/IosPower.kt` (stub).
- **Existing pattern to mirror:** the Android power impl in `:os-bridge`.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Battery-level / charging reporting — that's covered by [[09-ios-system-info]].
