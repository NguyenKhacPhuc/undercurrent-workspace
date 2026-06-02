---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/0
---

# [Substrate] The agent can share content via the iOS share sheet

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

"Share this" is a core handoff — text, a link, a file to another app. Works on Android; unimplemented on iOS.

## Acceptance criteria

- [ ] When the agent shares text or a link on iOS, the native share sheet appears with that content.
- [ ] When the agent shares a file on iOS, the share sheet offers it to compatible apps.
- [ ] If the user dismisses the share sheet without choosing a target, the agent is told the share was cancelled.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/sharing/IosSharing.kt` (stub) — needs a helper to find the currently-presented view to attach the sheet to.
- **Existing pattern to mirror:** the Android sharing impl in `:os-bridge`.
- **Watch out for:** presenting UIKit UI from the SDK needs the host's current view context — keep that lookup self-contained so the host doesn't have to pass it in.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Choosing/picking files to share from (media picker) — deferred (see [[out-of-scope]]).
