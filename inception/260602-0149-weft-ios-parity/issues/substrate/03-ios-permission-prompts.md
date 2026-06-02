---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 90m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/0
---

# [Substrate] The agent can request and check OS permissions on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

Capabilities that touch protected resources (microphone for voice, later: location, camera, contacts) can't run until the user grants permission. On Android the SDK maps its permission concept to the OS prompt; on iOS this is unimplemented. Provide the iOS mapping so permission-gated capabilities work.

## Acceptance criteria

- [ ] The agent can ask iOS for a permission and the user sees the native grant/deny prompt.
- [ ] The agent can check whether a permission is currently granted without prompting.
- [ ] A denied permission is reported back clearly so the agent can explain it can't proceed.
- [ ] Asking for a permission already granted returns "granted" without re-prompting.
- [ ] A permission the SDK knows about but iOS has no equivalent for is reported as "not applicable" rather than crashing.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

> [!tip]
> Read `weft/CLAUDE.md`; permissions live in two places (the concept in `:contracts`, the platform mapping in `:os-bridge`).

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/permissions/IosPermissions.kt` (stub); the `Permission` concept is in `weft/contracts/…`.
- **Existing pattern to mirror:** the Android mapping `AndroidPermissions.toAndroidPermission` in `:os-bridge` — mirror its shape for iOS authorization domains.
- **Watch out for:** [[11-ios-voice-input]] depends on this (mic + speech authorization). The host must declare matching usage-description strings — see [[open-questions]] Q1.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Capabilities behind these permissions (camera, location, contacts) — deferred, see [[out-of-scope]]. This story only delivers the request/check mechanism plus the mic+speech domains needed by voice input.
