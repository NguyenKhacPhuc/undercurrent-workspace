---
type: issue
feature: html-mini-apps
lane: substrate
status: done
wave: 2
estimate: 60m
claimed-by: SteveCastalk
merged-pr: https://github.com/NguyenKhacPhuc/android-harness/pull/25
merged-at: 2026-06-05T08:21:54Z
blocked-by: 
  - "[[01-bridge-call-action]]"
  - "[[03-scope-gate]]"
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/done
  - wave/2
---

# [Substrate] A mini-app can't reach the network or escape except through approved actions

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 2 — I control what a mini-app can touch
**API contract section:** n/a (no BE)

## Why

The bridge is only safe if the *rest* of the surface is sealed: a mini-app must not be able to make its own network calls, reach other origins, or escape the sandbox — its only path to the outside is the approved-action bridge. This closes the side doors.

## Acceptance criteria

- [ ] A mini-app cannot make arbitrary network requests of its own; any network goes through an approved action (subject to the app's existing allowlist).
- [ ] A mini-app cannot navigate away or load other origins.
- [ ] These restrictions hold on both iOS and Android.

## Blocked by

- [[01-bridge-call-action]], [[03-scope-gate]] — the approved-action path must exist before everything else is sealed around it.

## Hints (non-binding)

- **Likely surface:** a content security policy + disabling direct/cross-origin network in the WebView; route any mini-app network through an allowlisted action gated by the existing host `NetworkPolicy`.
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- The set of offerable network actions (host: [[02-offerable-actions]]).
