---
type: issue
feature: html-mini-apps
lane: substrate
status: ready
wave: 1
estimate: 60m
blocked-by: 
  - "[[01-bridge-call-action]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/ready
  - wave/1
---

# [Substrate] A mini-app saves and reloads its own state

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Mini-apps that remember and feel native
**API contract section:** n/a (no BE)

## Why

A mini-app that resets every time it opens isn't a tool — it's a toy. Letting a mini-app persist its own small state (a counter, a draft, settings) across opens makes trackers, forms, and tools actually useful.

## Acceptance criteria

- [ ] A mini-app can save a chunk of its own state and read it back on the next open.
- [ ] One mini-app's state is isolated from another's.
- [ ] Reading state that was never saved returns an empty/default, not an error.

## Blocked by

- [[01-bridge-call-action]] — same bridge plumbing.

## Hints (non-binding)

- **Likely surface:** `window.weft.getState()/setState(obj)` over the bridge, persisted per mini-app id (the host supplies the storage; substrate exposes the API).
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- Where the host ultimately stores it (host catalog story) — define the contract here.
