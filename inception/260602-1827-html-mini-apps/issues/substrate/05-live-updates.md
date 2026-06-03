---
type: issue
feature: html-mini-apps
lane: substrate
status: in-progress
wave: 1
estimate: 60m
claimed-by: SteveCastalk
blocked-by: 
  - "[[01-bridge-call-action]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/in-progress
  - wave/1
---

# [Substrate] The app can push live updates into a running mini-app

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 4 — Live and interactive
**API contract section:** n/a (no BE)

## Why

Some mini-apps show changing data (a live price, a countdown driven by app state, streamed progress). Letting the app push updates *into* a running mini-app — not just respond to its calls — makes live, reactive mini-apps possible.

## Acceptance criteria

- [ ] A running mini-app can register to receive updates and react when the app pushes one.
- [ ] Updates stop arriving once the mini-app is closed.

## Blocked by

- [[01-bridge-call-action]] — extends the same bridge with a native→JS direction.

## Hints (non-binding)

- **Likely surface:** `window.weft.onData(callback)` + a native push (`evaluateJavaScript` to invoke the registered callback).
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- Specific live data sources (host wires those).
