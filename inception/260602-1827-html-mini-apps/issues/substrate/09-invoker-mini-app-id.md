---
type: issue
feature: html-mini-apps
lane: substrate
status: done
wave: 4
estimate: 60m
claimed-by: SteveCastalk
merged-pr: https://github.com/NguyenKhacPhuc/android-harness/pull/26
merged-at: 2026-06-05T08:24:32Z
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/done
  - wave/4
---

# [Substrate] Thread the mini-app id through `MiniAppActionInvoker.invoke`

**Lane:** substrate (`weft/`)
**PRD section:** [[PRD]] → Story 2 — I control what a mini-app can touch
**API contract section:** n/a (no BE)

## Why

`MiniAppActionInvoker.invoke(name, argsJson)` carries no mini-app id, but a
single `HtmlComponent` instance renders **every** mini-app (the id arrives
per-render via `props.miniAppId`). So the host can only wire
id-*independent* actions through a singleton invoker — today just
`http_fetch`. The host's per-mini-app `store_get` / `store_set` handlers
(built + unit-tested in [[02-offerable-actions]] / [[01-html-mini-apps]]
slice 2, via `miniAppActionInvoker(...)`) can't be routed to the right
mini-app's state from a singleton, so they stay **unwired**. The bridge
already knows the id; it just doesn't pass it down. Threading it through
unlocks gated, per-mini-app key-value storage.

## Acceptance criteria

- [ ] `MiniAppActionInvoker.invoke` receives the rendering mini-app's id
      (the substrate `MiniAppBridge` already holds it via `miniAppId` /
      `props.miniAppId`).
- [ ] A single registered `HtmlComponent` can route `store_get` /
      `store_set` to the correct mini-app's state, isolated per id.
- [ ] Those store actions are gated by the approved-scope set — denied
      (rejected Promise) when the action isn't approved.
- [ ] Existing `http_fetch` and ungated-bridge behavior is unchanged;
      the host's current `miniAppHttpInvoker` keeps compiling (additive
      change).

## Blocked by

- None. The bridge foundation ([[01-bridge-call-action]], [[03-scope-gate]],
  [[04-mini-app-state]]) is already merged.

## Hints (non-binding)

> [!warning] **Cross-repo.** The contract change lands in `weft/` and must
> be bumped into the workspace `weft` submodule before the host re-wire.
- `weft/compose-defaults/.../MiniAppBridge.kt`: add the id to the
  `MiniAppActionInvoker` functional interface and pass `miniAppId` from
  `MiniAppBridge.dispatch(call)` into `invoker.invoke(...)`.
- **Host re-wire (consuming step, `undercurrent/`):** swap
  `miniAppHttpInvoker` → the existing `miniAppActionInvoker(...)` (which
  already includes the `store_get`/`store_set` handlers) in
  `AppModule.kt` + `IosKoinModule.kt`, now that `invoke` carries the id.
- **Verify:** substrate per `weft/CLAUDE.md`; host
  `./gradlew :feature:miniapps:test :<app>:compileDebugKotlin :composeApp:compileKotlinIosSimulatorArm64`.

## Out of scope for this story

- New offerable actions beyond making the existing `store_*` live.
- The HTTP allowlist tightening — that's [[05-tighten-miniapp-http-allowlist]].
