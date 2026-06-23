---
type: issue
feature: live-activity
lane: kmp-common
status: done
claimed-by: NguyenKhacPhuc
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/50
merged-at: 2026-06-23T09:22:38Z
wave: 0
estimate: 60m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/live-activity
  - status/done
  - wave/0
---

# [kmp-common] The waiting state is animated and alive instead of a frozen word

**Lane:** kmp-common
**PRD section:** [[PRD#Story 1 — The wait feels alive]]
**API contract section:** n/a (client-only)

## Why

The most direct fix for "it looks frozen and boring": replace the static "Thinking…" label with an animated indicator that, during quiet stretches, gently rotates through short hint lines. This alone makes every turn feel alive; later stories layer in what the assistant is specifically doing.

## Acceptance criteria

- [ ] As soon as a reply starts being prepared (and before its text arrives), an animated indicator appears where the static "Thinking…" used to be.
- [ ] During a quiet stretch — waiting on the model, or a gap between actions — the indicator rotates through a small set of short hint lines rather than holding one frozen word.
- [ ] The hints begin only after a brief quiet threshold, change at a calm cadence, and transition smoothly (no rapid flicker or jumpiness).
- [ ] When the reply's text begins streaming, the indicator yields to the streaming reply; the existing streaming/typing behavior is unchanged.
- [ ] The indicator and its hints appear only for visible chat replies — never for off-the-record turns.

## Blocked by

- nothing — independently grabbable.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`). Animation is Compose Multiplatform in `commonMain`.

- **Likely files affected:** the static thinking label at `ChatScreen.kt:151` and the chat feature's loading components (the existing `BlinkingCursor` is the precedent for a small in-chat animation in `commonMain`).
- **Watch out for:** the quiet threshold + cadence are tuning values ([[open-questions#Q1]]) — make them easy to adjust. Protect the off-the-record case from the start (the indicator must key off the visible in-flight state only).

## Out of scope for this story

- Narrating the *specific* action in progress (next story [[03-live-tool-narration]]).
- The after-turn persistent trail ([[04-friendly-step-trail]]).
