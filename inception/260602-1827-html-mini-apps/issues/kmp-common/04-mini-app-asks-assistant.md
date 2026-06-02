---
type: issue
feature: html-mini-apps
lane: kmp-common
status: ready
wave: 3
estimate: 60m
blocked-by: 
  - "[[07-ask-assistant-hook]]"
  - "[[01-html-mini-apps]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/ready
  - wave/3
---

# [kmp-common] A mini-app's "ask the assistant" runs a real agent turn

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 4 — Live and interactive
**API contract section:** n/a (no BE)

## Why

The substrate exposes a way for a mini-app to hand a request to the assistant, but it's inert until the host wires it to a real agent turn. This connects it, so a mini-app can genuinely ask the assistant to do something and react to the result.

## Acceptance criteria

- [ ] A mini-app's request to the assistant runs a real agent turn and the result comes back to the mini-app.
- [ ] The turn is auditable (recorded in traces) per [[open-questions]] Q2.
- [ ] If a mini-app sends a request and the user has no usable provider/key, it gets a clear failure, not a hang.

## Blocked by

- [[07-ask-assistant-hook]] — the substrate hook this fills in (via the bumped `weft` submodule).
- [[01-html-mini-apps]] — needs a runnable HTML mini-app to host the call.

## Hints (non-binding)

- **Likely files affected:** wire the substrate's assistant-request handler to the host chat/agent layer; decide foreground-vs-silent per Q2 (guess: silent + traced).
- **Verify (from `undercurrent/`):** `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- Surfacing the turn in the visible chat thread (deferred per Q2 unless the mob wants it).
