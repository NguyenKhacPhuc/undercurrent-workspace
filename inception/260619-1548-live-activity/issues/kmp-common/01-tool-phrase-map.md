---
type: issue
feature: live-activity
lane: kmp-common
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/live-activity
  - status/in-progress
  - wave/0
---

# [kmp-common] Every assistant action has a friendly description (never a raw name)

**Lane:** kmp-common
**PRD section:** [[PRD#Story 2 — See what the assistant is doing]]
**API contract section:** n/a (client-only)

## Why

Foundation for both the live indicator and the step trail: given an action the assistant takes, the app can describe it in plain language — a present-tense "doing" form, a past-tense "done" form, a "failed" form, and an icon. Critically, an unrecognized action must still read as plain language, never as a raw technical name. This story has no screen of its own; its observable contract is that translation.

## Acceptance criteria

- [ ] Each action the assistant can take today resolves to a friendly present-tense phrase, a past-tense phrase, a failure phrase, and an icon (e.g. opening the map → "Looking at the map…" / "Looked at the map" / "Couldn't reach the map").
- [ ] An action the app does not recognize — including ones a mini-app introduces — resolves to a sensible generic phrase set (e.g. "Working…" / "Done" / "Something went wrong") and an icon, and **never** exposes the raw technical action name.
- [ ] The descriptions read as plain language a non-technical person would understand — no underscores, no code-style names.

## Blocked by

- nothing — independently grabbable.

## Hints (non-binding)

> [!tip]
> Verify must include **both** target compiles (`compileDebugKotlinAndroid` + `compileKotlinIosSimulatorArm64`) plus the module's `test` task — see `undercurrent/CLAUDE.md`. The translation logic is pure and a natural fit for unit tests, including the unknown-action fallback.

- **Existing pattern to mirror:** the app's existing tool/action set and where actions are registered app-side (the same place the offerable actions + tool catalog live). The mapping belongs next to that, since the app owns its actions.
- **Watch out for:** the fallback is the safety net for the open-ended action set — test it explicitly with a made-up action name. Final copy/tone is a separate content pass ([[open-questions#Q2]]); land a sensible starter set here.

## Out of scope for this story

- Any UI — the indicator and trail consume this in later stories.
- The dead-air hint copy (that belongs to the indicator story, not the per-action map).
