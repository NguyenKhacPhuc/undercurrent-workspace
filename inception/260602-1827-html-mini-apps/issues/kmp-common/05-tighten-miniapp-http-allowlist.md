---
type: issue
feature: html-mini-apps
lane: kmp-common
status: ready
wave: 4
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/ready
  - wave/4
---

# [kmp-common] Constrain mini-app `http_fetch` to a real allowlist

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 2 — I control what a mini-app can touch
**API contract section:** n/a (no BE)

## Why

[[01-html-mini-apps]] slice 3 wired the `http_fetch` offerable action to a
**dedicated** mini-app `HttpClient` (deliberately not the BE-authenticated
one, so no auth-token leak), but with `NetworkPolicy.OPEN` — matching the
app's current posture. The offerable action is described as *"Fetch from
an allowlisted URL"*, so a script-enabled mini-app can today reach any
host, which the policy layer is supposed to prevent. The allowlist seam
already exists (the client installs weft's `NetworkAllowlistPlugin`); this
story replaces `OPEN` with a real, enforced allowlist.

## Acceptance criteria

- [ ] A script-enabled mini-app's `http_fetch` is constrained by an
      allowlist, not `NetworkPolicy.OPEN`.
- [ ] A fetch to a non-allowlisted host is refused before the network
      sees it, and the mini-app gets a clear rejected-Promise error.
- [ ] The allowlist is configurable (a default set + user additions)
      without a code change to add a host.
- [ ] No regression to the rest of the app's networking — this scopes the
      *mini-app* client only.

## Blocked by

- None. Builds on [[01-html-mini-apps]] (done) — the dedicated mini-app
  `HttpClient` it stands up.

## Hints (non-binding)

- The mini-app client is built in `AppModule.kt` (Android) +
  `IosKoinModule.kt` (iOS) via `whitelistingHttpClient(engine, policy)`.
  Swap `NetworkPolicy.OPEN` for a real `NetworkPolicy` (core allowlist +
  user-added hosts via `withUserAddition`).
- Decide where the user curates it — likely a settings surface or a
  per-mini-app grant; capture the choice in [[decisions]].
- **Verify (from `undercurrent/`):** `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- Per-mini-app (vs app-global) network grants, unless trivial to land here.
- Threading the mini-app id through the invoker — that's
  [[09-invoker-mini-app-id]].
