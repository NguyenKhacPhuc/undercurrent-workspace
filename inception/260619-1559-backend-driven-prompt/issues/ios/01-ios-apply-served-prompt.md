---
type: issue
feature: backend-driven-prompt
lane: ios
status: ready
wave: 1
estimate: 40m
blocked-by: ["[[01-prompt-config-repository]]"]
tags:
  - inception/issue
  - lane/ios
  - feature/backend-driven-prompt
  - status/ready
  - wave/1
---

# [iOS] iOS builds the assistant from the served prompt, not the baked constant

**Lane:** iOS
**PRD section:** [[PRD#Story 1 — The assistant runs on the served prompt]]
**API contract section:** consumes the provided prompt from [[01-prompt-config-repository]]

## Why

The iOS counterpart: assemble the assistant with the backend-provided base prompt instead of a compiled-in one, so the same central change reaches iOS users on next launch and behavior matches Android.

## Acceptance criteria

- [ ] On iOS, the assistant's replies use the backend-provided base prompt, not a compiled-in constant.
- [ ] When a newer prompt has been fetched, relaunching the app produces replies that reflect it.
- [ ] The assistant is not assembled with a blank or placeholder prompt — it is only built once a provided prompt exists.
- [ ] For the same served prompt, iOS behavior matches Android.

## Blocked by

- [[01-prompt-config-repository]] — reads the provided prompt from it.

## Hints (non-binding)

> [!tip]
> Verify per `undercurrent/CLAUDE.md` (iOS lane).

- **Likely files affected:** the iOS app's runtime/assistant bootstrap (the iOS host wires its own engine factory) where the base prompt is supplied; swap the source from a constant to the repository.
- **Watch out for:** same deferred-construction concern as Android — don't build the assistant until the provided prompt exists; coordinate with the cold-start gate ([[02-cold-start-gate]]).

## Out of scope for this story

- The Android equivalent ([[01-android-apply-served-prompt]]).
- The gate UI itself (kmp-common).
