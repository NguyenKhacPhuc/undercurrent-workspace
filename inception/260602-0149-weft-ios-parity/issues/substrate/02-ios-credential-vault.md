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

# [Substrate] Credentials persist securely on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

## Why

The agent stores secrets (provider keys, sign-in tokens). On Android these persist in secure device storage. On iOS this capability is unimplemented in the SDK — the undercurrent host wrote its own copy. Provide it in the SDK so every iOS host gets secure credential storage for free, and the host's copy can be deleted.

## Acceptance criteria

- [ ] A secret the agent stores on iOS can be read back unchanged.
- [ ] A stored secret survives an app restart.
- [ ] Removing a secret makes it no longer readable.
- [ ] Reading a secret that was never stored reports "not present" rather than failing.
- [ ] Secrets are held in the device's secure storage, not in plain app files.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

> [!tip]
> Read `weft/CLAUDE.md` and open the referenced files before editing.

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/keyvault/IosKeyVault.kt` (currently a stub).
- **Existing pattern to mirror:** undercurrent already has a working iOS impl at `undercurrent/core/domain/src/iosMain/kotlin/dev/weft/undercurrent/core/domain/KeychainKeyVaultRepository.kt` — lift it down. The host copy is deleted later in [[01-host-adopt-substrate-capabilities]].
- **Watch out for:** [[12-ios-provider-signin]] depends on this for token storage. Keep the stored-value semantics identical to the Android impl so host code is platform-agnostic.
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Wiring this into the one-call setup — that's [[14-ios-one-call-setup]].
