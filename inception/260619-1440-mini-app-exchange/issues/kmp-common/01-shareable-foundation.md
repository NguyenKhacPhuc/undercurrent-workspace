---
type: issue
feature: mini-app-exchange
lane: kmp-common
status: ready
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mini-app-exchange
  - status/ready
  - wave/0
---

# [kmp-common] A mini-app can carry sharing details and be installed from a shared bundle

**Lane:** kmp-common
**PRD section:** [[PRD#Story 3 — Install from a preview]]
**API contract section:** [[api-contract#The share bundle]]

## Why

Foundation slice for both directions of the exchange. A mini-app needs to remember a short description, that it was installed from a shared link, and who shared it — and the app needs one shared way to take a received bundle and turn it into a locally-installed mini-app that starts un-approved. Everything else (share UI, preview, revoke) builds on this.

This story has no screen of its own; its observable contract is the shared capability the other stories consume.

## Acceptance criteria

- [ ] A mini-app can hold an optional short description.
- [ ] A mini-app can record that it was installed from a shared link, including who shared it.
- [ ] The app can install a mini-app from a received share bundle (name, icon, description, HTML, declared capabilities), producing a normal local HTML mini-app.
- [ ] A mini-app installed from a bundle starts with **no** capabilities approved and **not yet** consented — so its first launch triggers the existing approval prompt.
- [ ] When installing, any declared capability that this app does not offer is dropped before it can ever be approved — the installed mini-app can never request a capability the app does not sanction.
- [ ] Installing the same bundle twice yields two independent local mini-apps, each with its own state and its own first-launch approval.

## Blocked by

- nothing — independently grabbable (works against the bundle shape in [[api-contract]]; no backend needed to build/test this slice).

## Hints (non-binding)

> [!tip]
> Verify must include **both** target compiles (`compileDebugKotlinAndroid` + `compileKotlinIosSimulatorArm64`) plus the module's `test` task — see `undercurrent/CLAUDE.md`. This is how we catch an Android-only API leaking into commonMain.

- **Existing pattern to mirror:** how HTML mini-apps are created today (the existing "create HTML mini-app" path that takes html + declared scopes and resets consent), and the existing capability-clamp that screens declared scopes against the host's offerable actions. Reuse that clamp — do not reimplement it.
- **Watch out for:** the un-consented / capability-clamp behavior is the safety invariant of the whole feature. It already exists for locally-authored HTML mini-apps; installing-from-bundle must route through the *same* path, not a parallel one.

## Out of scope for this story

- The Share UI, the preview screen, network calls — all in later stories.
- Storage backend choice (mini-apps persist the same way they do today).
