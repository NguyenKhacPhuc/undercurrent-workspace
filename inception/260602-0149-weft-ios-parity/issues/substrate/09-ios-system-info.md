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

# [Substrate] The agent can read device and system info on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

The agent tailors its behaviour to the device — name, OS version, locale, timezone, battery/charging state, network reachability, available memory. Works on Android; unimplemented on iOS.

## Acceptance criteria

- [ ] On iOS the agent can read device name, OS version, locale, and timezone.
- [ ] On iOS the agent can read battery level and whether the device is charging.
- [ ] On iOS the agent can tell whether the device currently has network connectivity.
- [ ] Each value reports a sensible "unknown" rather than crashing when iOS doesn't expose it.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/systeminfo/IosSystemInfo.kt` (stub).
- **Existing pattern to mirror:** the Android system-info impl, and the device-snapshot logic already present in `weft/runtime/src/iosMain/…DeviceSnapshot.ios.kt` (overlapping data — reuse, don't duplicate).
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Carrier/telephony info and wifi SSID — deferred (see [[out-of-scope]]).
