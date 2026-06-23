---
type: issue
feature: backend-driven-prompt
lane: android
status: ready
wave: 1
estimate: 40m
blocked-by: ["[[01-prompt-config-repository]]"]
tags:
  - inception/issue
  - lane/android
  - feature/backend-driven-prompt
  - status/ready
  - wave/1
---

# [Android] Android builds the assistant from the served prompt, not the baked constant

**Lane:** Android
**PRD section:** [[PRD#Story 1 — The assistant runs on the served prompt]]
**API contract section:** consumes the provided prompt from [[01-prompt-config-repository]]

## Why

On Android, the assistant is currently assembled with a compiled-in base prompt constant. This story swaps that for the backend-provided prompt, so changes made centrally take effect on next launch.

## Acceptance criteria

- [ ] On Android, the assistant's replies use the backend-provided base prompt, not the compiled-in constant.
- [ ] When a newer prompt has been fetched, relaunching the app produces replies that reflect it.
- [ ] The assistant is not assembled with a blank or placeholder prompt — it is only built once a provided prompt exists.
- [ ] With the seeded initial prompt, behavior at cut-over is identical to today's.

## Blocked by

- [[01-prompt-config-repository]] — reads the provided prompt from it.

## Hints (non-binding)

> [!tip]
> Verify per `undercurrent/CLAUDE.md` (Android lane). Force-stop after wiring changes so the runtime rebuilds.

- **Likely files affected:** the Android app's runtime/assistant wiring where the base prompt is passed in today (the engine already accepts an injected base prompt; this swaps the *source* of that string from a constant to the repository).
- **Watch out for:** the assistant is constructed eagerly today; with no fallback it must not be built until the provided prompt exists. Coordinate with the cold-start gate ([[02-cold-start-gate]]) so construction is deferred until the prompt is ready. Remove the now-dead baked constant as the source of truth ([[decisions#D4]]).

## Out of scope for this story

- The iOS equivalent ([[01-ios-apply-served-prompt]]).
- The gate UI itself (kmp-common).
