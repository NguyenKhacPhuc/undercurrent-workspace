---
type: issue
feature: html-mini-apps
lane: kmp-common
status: in-progress
claimed-by: SteveCastalk
wave: 2
estimate: 45m
blocked-by: 
  - "[[03-scope-gate]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/in-progress
  - wave/2
---

# [kmp-common] The app controls which actions are offerable to mini-apps

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 2 — I control what a mini-app can touch
**API contract section:** n/a (no BE)

## Why

The user approves a mini-app's requested actions from a menu of *offerable* actions. The app maintainer decides what's ever on that menu — so a mini-app can never even request something the app hasn't sanctioned. This is the policy layer above the substrate's enforcement.

## Acceptance criteria

- [ ] The app defines a set of actions that are offerable to mini-apps; a mini-app can only ever request from that set.
- [ ] A mini-app requesting an action that isn't offerable is rejected before it ever reaches the user for approval.
- [ ] The v1 offerable set is read-mostly (per [[open-questions]] Q1); destructive/sensitive actions are excluded.

## Blocked by

- [[03-scope-gate]] — builds on the substrate's per-mini-app approved-scope concept (consumed via the bumped `weft` submodule).

## Hints (non-binding)

- **Likely files affected:** a host-side action registry / allowlist that maps offerable action names to the underlying tools/capabilities, consumed when composing the mini-app runtime.
- **Verify (from `undercurrent/`):** `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- The approval screen ([[03-approve-on-first-run]]); per-action confirm friction (parked in Q1).
