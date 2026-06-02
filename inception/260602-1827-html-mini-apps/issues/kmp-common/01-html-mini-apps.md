---
type: issue
feature: html-mini-apps
lane: kmp-common
status: ready
wave: 2
estimate: 75m
blocked-by: 
  - "[[01-bridge-call-action]]"
  - "[[03-scope-gate]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/ready
  - wave/2
---

# [kmp-common] A mini-app can be a flexible HTML doc, saved for one-tap reuse

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 5 — Save it as a mini-app
**API contract section:** n/a (no BE)

## Why

Today a saved mini-app is a trigger prompt that re-runs the agent and caches a native render tree. This lets a mini-app instead *be* a flexible HTML document (with its declared actions and its own state) — saved, cached, and rendered on tap, instantly, without re-running the agent.

## Acceptance criteria

- [ ] A flexible (HTML) mini-app can be saved and reopened with one tap, rendering instantly from cache.
- [ ] Its declared actions and its saved state come back with it.
- [ ] Saved HTML mini-apps and the existing native-render mini-apps coexist in the same list.

## Blocked by

- [[01-bridge-call-action]], [[03-scope-gate]] — needs the substrate bridge + scope concept (consumed via the bumped `weft` submodule).

## Hints (non-binding)

> [!warning] **Cross-repo.** The substrate blockers must land in `weft/` and be bumped into the workspace `weft` submodule first.
- **Likely files affected:** the mini-app model + repository + the miniapps feature (`undercurrent/core/model`, `core/domain`, `feature/miniapps`) — extend the saved shape to carry an HTML doc + declared scopes + a state blob; the renderer picks HTML vs native-tree.
- **Verify (from `undercurrent/`):** `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- The approval UX ([[03-approve-on-first-run]]) and agent-turn wiring ([[04-mini-app-asks-assistant]]).
