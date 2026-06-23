---
type: issue
feature: backend-driven-prompt
lane: kmp-common
status: ready
wave: 1
estimate: 50m
blocked-by: ["[[01-prompt-config-repository]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/backend-driven-prompt
  - status/ready
  - wave/1
---

# [kmp-common] The assistant waits for a prompt before it's usable (no fallback)

**Lane:** kmp-common
**PRD section:** [[PRD#Story 2 — Offline and cold-start behavior]]
**API contract section:** [[api-contract#`GET /v1/prompt-config` — fetch the current base prompt]]

## Why

The user-facing consequence of "no compiled-in fallback": when there's no prompt to run on yet, the app must wait and tell the user clearly, rather than silently using a built-in prompt. Once a prompt is available (cached or freshly fetched), the assistant becomes usable.

## Acceptance criteria

- [ ] When a previously fetched prompt exists, the assistant is usable immediately, even offline.
- [ ] On a fresh start with no fetched prompt yet, the app shows a clear "getting set up" state and does not let the user start a conversation.
- [ ] If the fetch can't complete (no connection, server down), the state changes to a clear "couldn't connect — retry" message.
- [ ] Regaining connection, or retrying, completes the fetch and makes the assistant usable without a manual app restart.
- [ ] This gate happens after sign-in, in the normal startup flow (the user signs in first, then this).

## Blocked by

- [[01-prompt-config-repository]] — needs the "provided prompt / not-ready" signal.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`). The gate UI is Compose Multiplatform in `commonMain`.

- **Existing pattern to mirror:** the existing blocking sign-in gate on launch — this gate sits just after it and behaves similarly (block the main UI until a precondition is met).
- **Watch out for:** exact timing/copy/retry behavior is a parked decision ([[open-questions#Q1]]) — keep those tunable. Coordinate ordering with the per-platform stories: the assistant must not be constructed/usable until the provided prompt exists.

## Out of scope for this story

- Building the assistant from the prompt (per-platform stories).
- The server-side invalid-prompt guard (backend).
