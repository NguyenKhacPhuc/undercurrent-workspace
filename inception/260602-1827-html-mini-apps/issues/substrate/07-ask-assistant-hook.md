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

# [Substrate] A mini-app can hand a request to the assistant

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 4 — Live and interactive
**API contract section:** n/a (no BE)

## Why

Beyond fixed actions, a mini-app may need the assistant itself — "summarize this", "plan from these inputs". Exposing a way for a mini-app to fire an assistant request and receive the result lets mini-apps be agent-powered, not just scripted.

## Acceptance criteria

- [ ] A mini-app can send a request to the assistant and asynchronously receive the assistant's result.
- [ ] If no assistant request handler is wired by the host, the call returns a clear "not available" rather than hanging.

## Blocked by

- [[01-bridge-call-action]] — same bridge; this is a distinguished call routed to a host-provided handler.

## Hints (non-binding)

- **Likely surface:** `window.weft.sendMessage(text) -> Promise` routed to a host callback (the host wires it to a real agent turn in [[04-mini-app-asks-assistant]]). Whether the turn is foreground or silent is [[open-questions]] Q2.
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- Running the actual agent turn (host: [[04-mini-app-asks-assistant]]).
