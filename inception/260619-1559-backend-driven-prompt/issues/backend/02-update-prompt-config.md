---
type: issue
feature: backend-driven-prompt
lane: backend
status: ready
wave: 1
estimate: 40m
blocked-by: ["[[01-serve-prompt-config]]"]
tags:
  - inception/issue
  - lane/backend
  - feature/backend-driven-prompt
  - status/ready
  - wave/1
---

# [Backend] An authorized operator can change the served prompt

**Lane:** Backend
**PRD section:** [[PRD#Story 3 — Changing the prompt centrally]]
**API contract section:** [[api-contract#`PUT /v1/prompt-config` — replace the current base prompt (operator)]]

## Why

This is the payoff of the whole feature: the team can change the assistant's base prompt centrally and have clients pick it up on next launch — no app release.

## Acceptance criteria

- [ ] An authorized operator can replace the current base prompt with new text.
- [ ] After a successful change, the next fetch returns the new text with a bumped revision and updated time.
- [ ] A caller who is not an authorized operator cannot change the prompt.
- [ ] An empty or whitespace-only prompt is rejected, not stored (there is no client fallback, so a broken prompt must never be served).

## Blocked by

- [[01-serve-prompt-config]] — needs the config store + revision marker.

## Hints (non-binding)

- **Watch out for:** operator authorization mechanism is a parked decision — see [[open-questions#Q3]]. The invalid-prompt guard is the safety net for the no-fallback design ([[open-questions#Q2]]); at minimum reject empty/too-short.

## Out of scope for this story

- An editor/admin UI (out of scope for v1) and prompt history/rollback.
