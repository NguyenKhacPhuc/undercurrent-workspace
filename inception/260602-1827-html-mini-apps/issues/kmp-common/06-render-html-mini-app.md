---
type: issue
feature: html-mini-apps
lane: kmp-common
status: in-progress
wave: 3
estimate: 45m
claimed-by: SteveCastalk
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/in-progress
  - wave/3
---

# [kmp-common] Tap a saved HTML mini-app → render its document instantly

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 5 — Save it as a mini-app
**API contract section:** n/a (no BE)

## Why

The foundation k01 promised but left unwired: invoking a saved mini-app
still only re-runs the agent on its trigger prompt. A *flexible (HTML)*
mini-app should instead render its saved `htmlDocument` **on tap,
instantly, without an agent turn** — through the bridged `Html`
component (slice 3), so its `window.weft` bridge is live and scope-gated.
This is the render path [[03-approve-on-first-run]] and
[[04-mini-app-asks-assistant]] gate / extend.

## Acceptance criteria

- [ ] Invoking a mini-app that has a saved `htmlDocument` renders that
      document via the bridged `Html` component (with its `miniAppId` +
      `runScripts`), shown instantly — no agent turn fired.
- [ ] Invoking a native (`ui_render`) mini-app keeps the existing
      agent-turn behavior unchanged.
- [ ] The HTML render tree carries the mini-app's id so the bridge's
      scope gate + state store resolve to that mini-app.

## Blocked by

- None. Builds on k01 (persistence + bindings) and slice 3 (the bridged
  `Html` component is registered in the palette), both on `main`.

## Hints (non-binding)

- A pure builder `htmlMiniAppRenderTree(miniApp): ComponentNode` (type
  `Html`, props `{html, miniAppId, runScripts}`) is the testable core;
  the invoke handler emits it via `runtime.uiBridge` instead of calling
  the agent.
- **Android-primary:** iOS renders via `StubUiBridgeRepository` today, so
  the on-tap HTML render lands on Android; iOS parity tracks the iOS
  agent/UI bring-up.
- **Verify (from `undercurrent/`):** `./gradlew :feature:miniapps:test :feature:miniapps:compileDebugKotlinAndroid :feature:miniapps:compileKotlinIosSimulatorArm64 :androidApp:compileDebugKotlin`

## Out of scope for this story

- The first-run consent prompt ([[03-approve-on-first-run]]).
- The assistant-turn-from-mini-app hook ([[04-mini-app-asks-assistant]]).
