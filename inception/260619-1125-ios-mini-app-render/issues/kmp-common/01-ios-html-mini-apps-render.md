---
type: issue
feature: ios-mini-app-render
lane: kmp-common
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 90m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ios-mini-app-render
  - status/in-progress
  - wave/0
---

# [kmp-common] Saved HTML mini-apps open and run on iOS, at parity with Android

**Lane:** kmp-common (shared orchestration + iOS host wiring)
**PRD section:** [[PRD]] Story 1
**API contract section:** n/a (no BE work)

## Why

iOS users can save HTML mini-apps but can't open them — tapping one
replays the trigger prompt through chat instead of rendering the
mini-app. This slice brings the existing Android mini-app behavior to
iOS so a saved HTML mini-app opens, asks for consent on first run,
renders, runs scope-gated, persists its state, and can be saved for
reuse — exactly as on Android.

## Acceptance criteria

- [ ] Tapping a saved HTML mini-app on iOS opens it and renders its HTML
      content on screen, instead of producing a chat reply.
- [ ] The first time an HTML mini-app that declares capabilities runs on
      iOS, the user is asked to approve those capabilities before it
      renders. Approving runs the mini-app with exactly the approved
      capabilities; denying runs it with none.
- [ ] An HTML mini-app that declares no capabilities renders directly on
      iOS, with no consent prompt.
- [ ] A rendered HTML mini-app on iOS can use the host actions it was
      granted (e.g. fetching allowed data, reading/writing its stored
      data, asking the assistant) and is refused any action it was not
      granted.
- [ ] A mini-app's stored data persists on iOS: closing and reopening the
      mini-app restores what it saved.
- [ ] An iOS user can save the current on-screen result as a reusable
      mini-app; reopening that saved mini-app paints instantly without
      re-running the agent.
- [ ] Revisiting a previously-opened mini-app on iOS restores its last
      rendered view.
- [ ] Android mini-app behavior — opening, consent, scope-gated actions,
      state, and save — is unchanged.

## Blocked by

- nothing — independently grabbable.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Read
> `undercurrent/CLAUDE.md` and open the actual files before editing.

- **The gap is host orchestration, not rendering.** The substrate iOS
  WebView component (`weft` `Embed.ios.kt`) and the host's iOS bridge
  bindings (`IosKoinModule`) are already built and wired. What's missing
  is the iOS side of the logic that turns a mini-app tap into
  consent → render → state → save.
- **Existing pattern to mirror / lift:** the Android orchestrator at
  `undercurrent/feature/miniapps/src/androidMain/.../internal/WeftMiniAppViewModel.kt`
  already implements all of this with no Android-specific logic. The
  expected move is to make it shared (`commonMain`) and have the iOS DI
  construct it, replacing the stub that currently delegates to
  `IosAppViewModel.dispatchMiniApp` (which no-ops the consent / HTML
  render / save / cache paths). The Android DI (`androidApp/.../AppModule.kt`)
  shows the construction recipe.
- **`WeftRuntime` is shared, not Android-only** — despite its
  `dev.weft.android` package name it lives in `weft/runtime/commonMain`
  with an iOS actual, so the lift needs no `weft` change.
- **Watch out for:** the kmp-common verify must compile **both** targets
  (`compileDebugKotlinAndroid` + `compileKotlinIosSimulatorArm64`) plus
  run the `:feature:miniapps` tests — that's how an accidental
  Android-only-API leak into `commonMain` gets caught. Preserve the
  Android unit test coverage that currently exercises the orchestrator.

## Out of scope for this story

- Native-component (non-HTML) mini-app interaction-state persistence
  (separate future substrate story — html-mini-apps Q4).
- Any new host action or change to the offerable capability set.
- Real-device certification (deferred — [[decisions]] D3; simulator is
  the bar here).
