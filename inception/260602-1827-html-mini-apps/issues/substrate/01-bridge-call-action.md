---
type: issue
feature: html-mini-apps
lane: substrate
status: done
claimed-by: SteveCastalk
merged-pr: https://github.com/NguyenKhacPhuc/android-harness/pull/18
merged-at: 2026-06-02T18:20:09Z
wave: 0
estimate: 75m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/done
  - wave/0
---

# [Substrate] A mini-app's script can call an app action and get the result

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 1 — A mini-app that does something
**API contract section:** n/a (no BE; client-side `window.weft` API defined here)

## Why

The HTML surface already renders agent-authored HTML/JS but is sealed off — its script can draw anything yet can't *do* anything. This opens the first door: a mini-app's script can invoke a named app/device action and receive the result back, turning the canvas into a runtime. This is the foundation every other bridge capability builds on.

## Acceptance criteria

- [ ] A mini-app's script can call a named action and asynchronously receive its result.
- [ ] A call to an action that fails returns a clear failure to the mini-app (it doesn't hang).
- [ ] A call to an unknown action returns a clear "no such action" rather than silently doing nothing.
- [ ] The round-trip works on both iOS and Android.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

> [!tip] Read `weft/CLAUDE.md`. Builds on the existing `HtmlComponent` (`:compose-defaults`).
- **Likely surface:** a `window.weft.callTool(name, args) -> Promise` JS shim injected into the WebView, a native message handler (`WKScriptMessageHandler` iOS / a `@JavascriptInterface` Android), and a host-supplied action-invoker the handler routes to. Confirm against the actual `Embed.*` files.
- **Watch out for:** matching async semantics across `WKWebView` (`evaluateJavaScript` to resolve the promise) and Android WebView; this is the cross-platform crux.
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- The scope/permission gate (that's [[03-scope-gate]]) — this story can wire a single trusted action for the demo.
- State, theming, live push, lifecycle, agent turns (their own stories).
