---
type: open-questions
feature: live-activity
created: 2026-06-19
tags:
  - inception/open-questions
  - feature/live-activity
---

# Open questions — Live Activity

> [!question] For the mob to resolve. Driver guesses give a starting point; none block Wave 0.

## Q1 — Dead-air timing: quiet threshold + rotation cadence

How long a quiet stretch before the rotating hints kick in, and how often do they change? **[DRIVER GUESS: hints start after ~1.5–2s of no new activity and no streaming text; rotate every ~3–4s; cross-fade, never hard-cut.]** Mob: confirm the feel — fast enough to feel alive, slow enough to not distract.

## Q2 — The actual copy (phrases + hints + failures)

This is a content/tone pass, not engineering:
- The friendly "doing / done / failed" phrase + icon for each of today's actions (open map, set theme, share, remember a fact, fetch from the web, build a mini-app, navigate, …).
- The small set of dead-air hint lines.
- The generic fallback phrasing for unrecognized actions.

**[DRIVER GUESS: Construction lands a sensible starter set inline; the mob does a tone pass before release.]** Mob: who owns final copy, and is there a brand voice to match? (And: should a "remember a fact"-style action with no visible effect still appear as a step, or stay silent?)

## Q3 — Trail visual density

How compact is the persistent trail? **[DRIVER GUESS: a single low-contrast line of icon + short past-tense labels, wrapping if long; no expand/collapse in v1; the detailed trace screen remains the place for depth.]** Mob: confirm one-line-compact is enough, or do tool-heavy turns want a tighter summary (e.g. "3 steps")?

## Q4 — Trail on actionless replies

A plain text reply with no actions: trail shown or not? **[DRIVER GUESS: no actions → no trail; just the answer. The live indicator still animated while waiting.]** Mob: confirm.
