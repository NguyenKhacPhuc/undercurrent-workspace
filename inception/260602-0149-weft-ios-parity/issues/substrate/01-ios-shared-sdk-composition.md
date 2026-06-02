---
type: issue
feature: weft-ios-parity
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 120m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/0
---

# [Substrate] iOS and Android assemble the SDK from one shared mechanism

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 2 — Shared setup, not duplicated
**API contract section:** n/a (no BE)

## Why

Today the SDK's composition — how all its pieces are wired together into a runnable whole — exists as an Android path and a separate, thinner iOS path. That split is why adding an iOS host means re-deriving the assembly. Move the shared assembly into common code so both platforms compose the SDK the same way, with only genuinely platform-bound pieces supplied per platform.

## Acceptance criteria

- [ ] Setting up the SDK on iOS and on Android goes through the same shared composition, not two parallel ones.
- [ ] An iOS host can stand up a runnable SDK through that shared composition and complete an agent turn on a real iOS device.
- [ ] Android host setup behaves exactly as before — no change in how an Android host composes or runs.
- [ ] The only things still supplied per platform are the genuinely platform-bound pieces (storage driver, networking engine, device snapshot) — everything else is shared.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

> [!tip]
> Hints orient Construction; they are not a contract. Read `weft/CLAUDE.md` and open the existing files before editing.

- **Likely files affected:** `weft/runtime/src/commonMain/…` (composition root), `weft/runtime/src/androidMain/…WeftRuntime.create(...)`, `weft/runtime/src/iosMain/kotlin/dev/weft/android/WeftRuntimeIos.kt` — confirm against actual code.
- **Existing pattern to mirror:** the Android `WeftRuntime.create(context)` path is the reference for what "fully wired" means; the iOS factory currently diverges from it.
- **Watch out for:** this underpins [[13-ios-debug-overlay]] (the overlay reads the composition) and [[14-ios-one-call-setup]]. Don't regress the Android composition while lifting it — that's the primary risk.
- **Verify (from `weft/`):** `./gradlew :runtime:compileKotlinIosSimulatorArm64` · `./gradlew :runtime:compileDebugKotlinAndroid` · `./gradlew :runtime:test` · `./gradlew detekt`

## Out of scope for this story

- Implementing any individual device capability (separate stories).
- A broader composition-root redesign beyond what cross-platform sharing needs (see [[out-of-scope]]).
