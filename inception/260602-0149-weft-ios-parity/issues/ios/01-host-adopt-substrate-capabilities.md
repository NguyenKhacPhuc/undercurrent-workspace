---
type: issue
feature: weft-ios-parity
lane: ios
status: superseded
wave: 3
estimate: 90m
blocked-by:
  - "[[14-ios-one-call-setup]]"
  - "[[02-ios-credential-vault]]"
  - "[[11-ios-voice-input]]"
tags:
  - inception/issue
  - lane/ios
  - feature/weft-ios-parity
  - status/ready
  - wave/3
---

# [iOS] Undercurrent adopts the common SDK setup and drops its duplicated impls

> [!warning] **Superseded (2026-06-02).** Construction found this is not "adoption only" — undercurrent's iOS host is a ~10-stub "coming-soon" shell (`StubAgentEngine` + stub repos), so adopting the substrate is a full **iOS agent bring-up**. Re-inceptioned as its own epic: `inception/260602-1335-ios-agent-bringup/` (8 sliced stories). The credential-vault + voice "lift & delete" pieces are folded into that feature's `02-share-secure-key-repo` (done-able now) and the deferred voice work.

**Lane:** iOS (`undercurrent/`)
**PRD section:** [[PRD]] → Story 7 — Undercurrent adopts the mechanism
**API contract section:** n/a (no BE)

## Why

Proof that the mechanism is genuinely common: switch undercurrent's iOS build from its hand-assembled SDK setup to the SDK's single setup call, and delete the in-app capability copies (secure credential storage, voice input) now that the SDK provides them. A copy left behind is duplication that drifts.

> [!warning] **Cross-repo dependency.** The blockers are substrate stories in `weft/`. They must land and be bumped into this workspace's `weft` submodule before this story is grabbable. This is the one cross-repo wait in the feature (see [[decisions]] D3).

## Acceptance criteria

- [ ] Undercurrent's iOS build composes the SDK through the single setup call, with no hand-assembled stand-in capability set.
- [ ] Undercurrent no longer carries its own iOS copies of secure credential storage or voice input — those now come from the SDK.
- [ ] The iOS app behaves the same as before for everything it already did (credentials persist, voice input works, provider sign-in works).
- [ ] The required iOS usage-description strings (microphone, speech recognition) are present so permission-gated capabilities work on a real device.
- [ ] The iOS app still builds and runs an agent turn on a real device after the migration.

## Blocked by

- [[14-ios-one-call-setup]] — the single setup call this story adopts.
- [[02-ios-credential-vault]] and [[11-ios-voice-input]] — the SDK impls that replace the deleted host copies.

## Hints (non-binding)

- **Likely files affected:** `undercurrent/composeApp/src/iosMain/…IosKoinModule.kt` (DI wiring → use the SDK setup call); delete `undercurrent/core/domain/src/iosMain/…KeychainKeyVaultRepository.kt` and `…IosSpeechRepository.kt` once the SDK provides them; the host's iOS app config for usage-description strings.
- **Existing pattern to mirror:** the Android host's SDK setup in undercurrent is the reference for "single setup call".
- **Watch out for:** `KeychainSessionTokenStore` may also be liftable, but that's not required by this feature — keep this story to the credential-vault + voice swap unless the SDK already covers it. Resolves [[open-questions]] Q1.
- **Verify (from `undercurrent/`):** see `undercurrent/CLAUDE.md` — iOS build/run commands, plus the shared-module compile checks.

## Out of scope for this story

- Any new user-facing app feature — this is adoption only.
- Lifting capabilities the SDK doesn't yet provide (the deferred backlog — see [[out-of-scope]]).
