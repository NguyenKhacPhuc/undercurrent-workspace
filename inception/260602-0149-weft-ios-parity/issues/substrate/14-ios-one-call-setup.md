---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 2
estimate: 60m
blocked-by:
  - "[[01-ios-shared-sdk-composition]]"
  - "[[02-ios-credential-vault]]"
  - "[[03-ios-permission-prompts]]"
  - "[[04-ios-clipboard]]"
  - "[[05-ios-open-links]]"
  - "[[06-ios-haptics]]"
  - "[[07-ios-screen-power]]"
  - "[[08-ios-image-editing]]"
  - "[[09-ios-system-info]]"
  - "[[10-ios-share-sheet]]"
  - "[[11-ios-voice-input]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/2
---

# [Substrate] One-call iOS setup, with graceful gaps

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 1 — One-call iOS setup · Story 4 — Graceful gaps
**API contract section:** n/a (no BE)

## Why

This is the payoff that makes the mechanism *common*. With the shared composition and the foundational capabilities in place, an iOS host should obtain a fully-wired SDK from a single setup call — no hand-assembly, no supplying its own stand-ins — exactly the shape Android offers. Capabilities still deferred on iOS must degrade cleanly so the agent never crashes reaching for one.

## Acceptance criteria

- [ ] An iOS host obtains a fully-wired device-capability set from a single setup call, with no custom stand-ins required.
- [ ] Used with no overrides, that call yields an SDK that runs an agent turn on a real iOS device with the foundational capabilities all working.
- [ ] A host can still override any individual capability if it wants to.
- [ ] When the agent invokes a capability not yet implemented on iOS, it receives a clear "not available on this device" result instead of crashing.
- [ ] The single setup call mirrors the Android one in shape, so host code reads the same across platforms.

## Blocked by

- All foundational capability stories + the shared composition — this assembles them. See frontmatter.

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/IosOsCapabilities.kt` (the composer) + a public factory entry point mirroring Android's `AndroidOsCapabilities.create(...)`.
- **Existing pattern to mirror:** `AndroidOsCapabilities.create(context)` is the reference for "one call → fully wired". Deferred capabilities default to an "unsupported on iOS" result (see [[out-of-scope]] for the deferred list).
- **Watch out for:** this is the integration point for the whole feature — it's intentionally the last substrate wave. Keep the unimplemented-capability fallback uniform so every deferred capability behaves consistently.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew :runtime:test` · `./gradlew detekt`

## Out of scope for this story

- Implementing any deferred capability — they only need to fall back cleanly here.
- The undercurrent migration — that's [[01-host-adopt-substrate-capabilities]] (ios lane).
