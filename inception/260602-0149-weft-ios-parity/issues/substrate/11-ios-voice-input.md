---
type: issue
feature: weft-ios-parity
lane: substrate
status: ready
wave: 1
estimate: 90m
blocked-by:
  - "[[03-ios-permission-prompts]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/weft-ios-parity
  - status/ready
  - wave/1
---

# [Substrate] Voice input works on iOS

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Foundational capabilities work on iOS
**API contract section:** n/a (no BE)

> [!warning] **Blocked by a toolchain gap (2026-06-02).** `Speech.recognize()` needs `AVAudioSession.setActive/setCategory/setMode`, which are **unresolved** in Kotlin/Native 2.3.10 vs the iOS 26.4 SDK (compile-probed in weft; undercurrent hit the same wall and its `IosSpeechRepository` is a no-op stub — so the "lift it" hint below is void). Deferred until a custom cinterop `.def` binds AVAudioSession + SFSpeechRecognizer, or a K/N release ships refreshed bindings. See [[open-questions]] Q3. `say`/`stop` (TTS) are unaffected.

## Why

Speaking to the agent is a primary input on mobile. On Android the SDK provides speech recognition; on iOS the undercurrent host wrote its own copy. Provide it in the SDK so every iOS host gets voice input, and the host copy can be deleted.

## Acceptance criteria

- [ ] On iOS the user can speak and have their words turned into text the agent receives.
- [ ] Before first use, iOS asks for microphone + speech-recognition permission; denial is reported clearly and the agent explains it can't listen.
- [ ] Recognition ends cleanly when the user stops speaking, and the recognised text is delivered.
- [ ] If the user cancels mid-listen, no partial result is forced on the agent and the agent is told it was cancelled.

## Blocked by

- [[03-ios-permission-prompts]] — needs microphone + speech authorization to be requestable.

## Hints (non-binding)

- **Likely files affected:** `weft/os-bridge/src/iosMain/kotlin/dev/weft/osbridge/speech/IosSpeech.kt` (stub).
- **Existing pattern to mirror:** undercurrent already has a working iOS impl at `undercurrent/core/domain/src/iosMain/…IosSpeechRepository.kt` — lift it down. The host copy is deleted in [[01-host-adopt-substrate-capabilities]].
- **Watch out for:** the host must declare the speech + microphone usage-description strings (see [[open-questions]] Q1).
- **Verify (from `weft/`):** `./gradlew :os-bridge:compileKotlinIosSimulatorArm64` · `./gradlew :os-bridge:compileDebugKotlinAndroid` · `./gradlew :os-bridge:test` · `./gradlew detekt`

## Out of scope for this story

- Text-to-speech / audio playback and recording — deferred (see [[out-of-scope]]).
