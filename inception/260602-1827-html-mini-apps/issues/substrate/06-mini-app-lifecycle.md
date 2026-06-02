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

# [Substrate] Mini-apps get open/close signals and several can run

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 4 — Live and interactive
**API contract section:** n/a (no BE)

## Why

A mini-app needs to know when it becomes visible or goes away (to start/stop timers, save state, release resources), and more than one mini-app may be open at once. Lifecycle signals + multi-instance isolation make mini-apps well-behaved.

## Acceptance criteria

- [ ] A mini-app is told when it opens and when it's about to close.
- [ ] Two mini-apps open at once don't interfere (state, callbacks, and theming stay per-instance).
- [ ] A closed mini-app stops receiving anything.

## Blocked by

- [[01-bridge-call-action]] — same bridge; lifecycle rides on the per-instance handler.

## Hints (non-binding)

- **Likely surface:** lifecycle callbacks on `window.weft` + per-instance message-handler scoping so callbacks/state don't cross instances.
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- Persisting state across the close (that's [[04-mini-app-state]]).
