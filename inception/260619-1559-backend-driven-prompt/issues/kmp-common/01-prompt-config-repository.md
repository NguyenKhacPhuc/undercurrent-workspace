---
type: issue
feature: backend-driven-prompt
lane: kmp-common
status: ready
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/backend-driven-prompt
  - status/ready
  - wave/0
---

# [kmp-common] The app fetches, caches, and provides the current base prompt

**Lane:** kmp-common
**PRD section:** [[PRD#Story 1 — The assistant runs on the served prompt]], [[PRD#Story 2 — Offline and cold-start behavior]]
**API contract section:** [[api-contract#`GET /v1/prompt-config` — fetch the current base prompt]]

## Why

Foundation for both platforms: a shared way to fetch the current base prompt from the backend, remember the last one fetched, and provide "the prompt to use right now" to whatever builds the assistant. No screen of its own; its observable contract is that provision.

## Acceptance criteria

- [ ] On request, the app fetches the current base prompt from the backend using the signed-in session.
- [ ] A successfully fetched prompt is stored locally and becomes the provided prompt.
- [ ] When offline, the most recently fetched prompt is provided.
- [ ] When no prompt has ever been fetched, the provided prompt is clearly absent (not an empty string and not a built-in default) — so callers can tell "not ready yet" apart from "the prompt is empty".
- [ ] A failed fetch (offline, server error) does not discard a previously fetched prompt.

## Blocked by

- nothing — independently grabbable (works against [[api-contract]] with a mock; no live backend needed to build/test this slice).

## Hints (non-binding)

> [!tip]
> Verify must include **both** target compiles (`compileDebugKotlinAndroid` + `compileKotlinIosSimulatorArm64`) plus the module's `test` task — see `undercurrent/CLAUDE.md`.

- **Existing pattern to mirror:** how the app already caches small bits of remote/profile state locally (DataStore-backed repositories) and how authenticated calls attach the session token.
- **Watch out for:** "never fetched" must be a distinct, explicit state — the no-fallback decision ([[decisions#D4]]) depends on callers being able to block on it rather than running with a blank prompt.

## Out of scope for this story

- The blocking cold-start UX ([[02-cold-start-gate]]).
- Wiring the prompt into the running assistant (per-platform stories).
