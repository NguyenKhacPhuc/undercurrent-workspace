---
type: issue
feature: html-mini-apps
lane: substrate
status: ready
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/html-mini-apps
  - status/ready
  - wave/0
---

# [Substrate] A mini-app adopts the app's look automatically

**Lane:** Substrate (`weft/`)
**PRD section:** [[PRD]] → Story 3 — Mini-apps that remember and feel native
**API contract section:** n/a (no BE)

## Why

A flexible HTML mini-app that ships with raw browser defaults looks foreign next to the native UI. Injecting the app's design tokens (colors, typography, spacing) so the HTML can style against them makes mini-apps feel like part of the app by default.

## Acceptance criteria

- [ ] A mini-app rendered with default styling reads as part of the app (uses the app's colors and typography).
- [ ] A mini-app's script/CSS can reference the app's theme tokens.
- [ ] The look updates correctly between light and dark.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely surface:** inject the host theme as CSS custom properties + a small base stylesheet into the rendered HTML; expose tokens via `window.weft.theme`.
- **Verify (from `weft/`):** `./gradlew :compose-defaults:compileKotlinIosSimulatorArm64 :compose-defaults:compileDebugKotlinAndroid` + `./gradlew detekt`

## Out of scope for this story

- The action bridge / state / etc. (their own stories).
