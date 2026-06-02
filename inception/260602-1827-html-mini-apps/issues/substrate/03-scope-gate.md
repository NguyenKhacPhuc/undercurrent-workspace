---
type: issue
feature: html-mini-apps
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 1
estimate: 75m
blocked-by: 
  - "[[01-bridge-call-action]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/in-progress
  - wave/1
---

# [Substrate] A mini-app can only reach actions it declared and were approved

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 2 — I control what a mini-app can touch
**API contract section:** n/a (no BE)

## Why

Mini-app code is agent-authored and semi-untrusted. The bridge is the boundary between that code and real device/app actions, so it must enforce that a mini-app only reaches the actions it *declared* and that were *approved* — anything else is refused. This is the security gate the whole permissioned model rests on ([[decisions]] D2).

## Acceptance criteria

- [ ] A mini-app calling an action it declared and that was approved succeeds.
- [ ] A mini-app calling an action it did NOT declare, or that was denied, is refused with a clear "not permitted" result.
- [ ] The set of approved actions is supplied to the runtime per mini-app (the substrate enforces; it does not decide policy).

## Blocked by

- [[01-bridge-call-action]] — the gate sits in front of the call path.

## Hints (non-binding)

- **Likely surface:** the message handler checks the call's action name against the per-mini-app approved-scope set before invoking; the host supplies that set. The *menu* of offerable actions + the approval UX + the grant store are host stories ([[02-offerable-actions]], [[03-approve-on-first-run]]).
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- The consent UX + grant persistence (host) and the offerable-action menu (host).
