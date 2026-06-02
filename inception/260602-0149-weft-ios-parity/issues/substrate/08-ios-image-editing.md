---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 60m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/0
---

# [Substrate] The agent can resize, crop, and rotate images on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

The agent often has to prepare an image — shrink it before sending, crop to a region, rotate to upright. Works on Android; unimplemented on iOS. No permissions involved, so it's a clean self-contained slice.

## Acceptance criteria

- [ ] The agent can resize an image on iOS and the result has the requested dimensions.
- [ ] The agent can crop an image to a region on iOS and the result contains only that region.
- [ ] The agent can rotate an image on iOS and the result is rotated as requested.
- [ ] Operating on unreadable image data reports a clear failure rather than crashing.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/imageops/IosImageOps.kt` (stub).
- **Existing pattern to mirror:** the Android image-ops impl in `:os-bridge` — match its input/output data shapes so callers are platform-agnostic.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Capturing or picking images (camera, media picker) — deferred (see [[out-of-scope]]).
