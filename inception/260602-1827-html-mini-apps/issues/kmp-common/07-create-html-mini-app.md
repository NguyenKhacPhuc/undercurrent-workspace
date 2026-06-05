---
type: issue
feature: html-mini-apps
lane: kmp-common
status: done
wave: 3
estimate: 45m
claimed-by: SteveCastalk
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/30
merged-at: 2026-06-05T09:49:05Z
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/done
  - wave/3
---

# [kmp-common] The agent can author + save an HTML mini-app (the entry point)

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 5 — Save it as a mini-app
**API contract section:** n/a (no BE)

## Why

The gap that made the whole feature unreachable: every story built the
*engine* (persistence, bridge, render-on-tap, consent, actions, assistant)
but nothing ever **created** an HTML mini-app. `MiniAppsRepository.setHtmlDocument`
had zero callers; `create_mini_app` only saves a native trigger-prompt
mini-app, so `htmlDocument` was always null and none of the machinery
fired. This adds the ignition.

## Acceptance criteria

- [ ] The agent can save a self-contained HTML document as a mini-app,
      declaring the actions it needs — persisted with `htmlDocument` +
      `declaredScopes` set.
- [ ] The saved HTML mini-app then flows through the existing path:
      renders on tap ([[06-render-html-mini-app]]), asks consent on first
      run ([[03-approve-on-first-run]]), runs scope-gated.
- [ ] Native (`create_mini_app`) mini-apps are unaffected.

## Hints (non-binding)

- `MiniAppsRepository.addHtml(name, emoji, html, declaredScopes)` + a
  `create_html_mini_app` agent tool (`:data:weft`, Android) registered in
  `AppModule`'s `extraToolsFactory`. Also: a one-line preamble nudge so
  the agent knows *when* to reach for an HTML mini-app vs a native
  component (resolves [[open-questions]] Q3).
- **Verify:** `:core:domain` both targets + `:data:weft:compileDebugKotlin`
  + `:androidApp:compileDebugKotlin`.

## Out of scope for this story

- A user-facing "save as HTML mini-app" affordance (this is agent-driven).
- iOS (the tool is Android, matching the Android-primary render path).
