---
type: decisions
feature: live-activity
created: 2026-06-19
tags:
  - inception/decisions
  - feature/live-activity
---

# Decisions — Live Activity

> [!info] Unilateral driver decisions to ratify in mob review, with rationale so the mob can challenge the *why*.

## D1 — Scope: animate + narrate + record; not persona, not reasoning

**Decision:** v1 delivers (a) an animated waiting indicator, (b) live friendly narration of the assistant's actions, and (c) a persistent friendly trail of the steps. It does **not** flavor the copy by persona, and does **not** surface the model's internal reasoning tokens.

**Why:** The animation+narration+trail trio fully answers the "it looks frozen and boring" problem using signal that already flows to the client. Persona flavor is pure polish and is cheap to add later (persona is already available in the turn). Reasoning tokens would require substrate work and a separate UX debate (how much private thinking to expose) — out of proportion for v1.

## D2 — The step record persists (not collapse-on-finish)

**Decision:** After a turn, the friendly steps stay attached to the reply as a visible trail, rather than collapsing away.

**Why:** Transparency — the user can always see how the assistant got to the answer, which builds trust in a tool-using agent. The cost is visual clutter, mitigated by a deliberately compact, low-key treatment (see [[open-questions#Q3]]).

## D3 — Dead air shows rotating playful hints (not a single generic word)

**Decision:** During genuinely quiet stretches (model latency before first token, gaps between actions), the indicator rotates through short hint lines.

**Why:** A single frozen word is exactly the boredom we're removing; gentle rotation keeps the surface alive. Guardrails matter: hints start only after a brief quiet threshold, rotate at a calm cadence, and must not flicker — otherwise "alive" becomes "jittery / gimmicky" (threshold + cadence in [[open-questions#Q1]]).

## D4 — Friendly phrases replace raw tool names; app-side map with a generic fallback

**Decision:** Each action the assistant can take maps to a friendly "doing", "done", and "failed" phrase plus an icon. The mapping lives in the app (which owns its action set). Any unrecognized action — including mini-app-authored or future ones — falls back to a sensible generic phrase, so a raw technical name is **never** shown.

**Why:** The narration is only reassuring if it reads like plain language. The fallback is non-negotiable because the action set is open-ended (mini-apps can introduce actions); without it, the feature would leak `snake_case` names. Exact copy is a content pass (see [[open-questions#Q2]]).

## D5 — Visible turns only; off-the-record turns stay silent

**Decision:** The live indicator and the step trail apply only to visible chat replies. Off-the-record turns (the ephemeral mini-app → assistant path shipped in `inception/260619-1338-ephemeral-assistant-turn/`) must not surface any activity in the visible chat.

**Why:** Those turns are intentionally invisible; narrating them would leak hidden activity into the transcript and regress that feature. This is a hard non-goal, called out so Construction protects it.
