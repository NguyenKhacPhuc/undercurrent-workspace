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

# [Substrate] The agent can open links on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

The agent frequently hands off to other apps by opening a URL (a map, a web page, a `tel:` link). Works on Android; unimplemented on iOS.

## Acceptance criteria

- [ ] When the agent opens a web link on iOS, the system opens it in the appropriate app.
- [ ] When the agent opens a link to another installed app, iOS switches to that app.
- [ ] Opening a link iOS can't handle reports back cleanly rather than crashing.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/intents/IosIntents.kt` (stub).
- **Existing pattern to mirror:** the Android intents/openUrl impl in `:os-bridge`.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Deep links into specific iOS Settings panels — deferred (see [[out-of-scope]]).
