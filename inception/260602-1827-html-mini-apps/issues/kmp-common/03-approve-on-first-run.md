---
type: issue
feature: html-mini-apps
lane: kmp-common
status: ready
wave: 3
estimate: 75m

blocked-by: 
  - "[[01-html-mini-apps]]"
  - "[[02-offerable-actions]]"
  - "[[03-scope-gate]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/html-mini-apps
  - status/ready
  - wave/3
---

# [kmp-common] The user approves what a mini-app can access, on first run

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 2 — I control what a mini-app can touch
**API contract section:** n/a (no BE)

## Why

The whole permissioned model is only legible if the user *sees* it: the first time a mini-app wants device/app actions, they get a clear "this mini-app wants to: …" prompt and approve or deny. The choice is remembered and changeable. This is the consent UX that makes flexible mini-apps trustworthy.

## Acceptance criteria

- [ ] The first time a mini-app needs actions, the user sees a plain-language list of what it's asking for and approves or denies.
- [ ] After approval, the mini-app can use exactly those actions and no more.
- [ ] After denial, the mini-app runs but its blocked calls fail clearly (it doesn't silently misbehave).
- [ ] The user can review and change a mini-app's granted actions later.

## Blocked by

- [[01-html-mini-apps]], [[02-offerable-actions]] — needs HTML mini-apps + the offerable menu.
- [[03-scope-gate]] — the substrate enforces the approved set this UX produces (via the bumped `weft` submodule).

## Hints (non-binding)

- **Likely files affected:** a consent screen/sheet + a per-mini-app grant store; feed the approved set into the mini-app runtime composition.
- **Verify (from `undercurrent/`):** `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- Per-use confirmation for destructive actions (parked in [[open-questions]] Q1).
