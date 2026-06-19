---
type: issue
feature: ephemeral-assistant-turn
lane: kmp-common
status: in-progress
claimed-by: SteveCastalk
wave: 1
estimate: 30m
blocked-by: ["[[01-isolated-one-shot-turn]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ephemeral-assistant-turn
  - status/in-progress
  - wave/1
---

# [kmp-common] A mini-app's question to the assistant stays out of the user's chat

**Lane:** kmp-common (host — both platforms)
**PRD section:** [[PRD]] Story 2
**API contract section:** n/a (no BE work)

## Why

Mini-apps can ask the assistant on both platforms, but the exchange lands
in the user's chat thread and saved history. This slice routes that
question through the new isolated turn so the user's conversation stays
clean — the last v1 caveat from the mini-app features.

## Acceptance criteria

- [ ] After a mini-app asks the assistant, the question and answer do not
      appear in the chat thread on screen.
- [ ] The exchange does not appear in the conversation list / saved
      history.
- [ ] The mini-app still receives the assistant's answer and keeps
      working.
- [ ] If the assistant isn't ready, the mini-app's request fails
      gracefully (unchanged from today — no crash, no hang).
- [ ] Behaves identically on Android and iOS.

## Blocked by

- [[01-isolated-one-shot-turn]] — needs the substrate isolated-turn
  capability. (Can develop against the `weft` branch before it merges;
  see [[decisions]] D3.)

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Read
> `undercurrent/CLAUDE.md` and open the actual files before editing.

- **Single touch-point:** the shared `askAssistant(agent, text)` in
  `undercurrent/feature/chat/.../agent/MiniAppAssistant.kt` currently runs
  a normal turn (`dispatchAndAwait(Send)`) and reads the reply from the
  agent's history. Switch it to the substrate isolated turn from
  [[01-isolated-one-shot-turn]], which returns the reply directly — both
  the Android and iOS handlers already call `askAssistant`, so one change
  covers both platforms.
- **The reply path simplifies:** the isolated turn returns the text, so
  the `lastAssistantText(history)` read is no longer needed for this call.
- **Watch out for:** kmp-common verify must compile **both** targets
  (`compileDebugKotlinAndroid` + `compileKotlinIosSimulatorArm64`) plus
  run the chat-module tests. Update/replace the existing
  `MiniAppAssistant` tests to reflect the isolated-turn behavior.

## Out of scope for this story

- The substrate capability itself ([[01-isolated-one-shot-turn]]).
- Memory/tool-effect isolation ([[decisions]] D2).
