---
type: out-of-scope
feature: live-activity
created: 2026-06-19
tags:
  - inception/out-of-scope
  - feature/live-activity
---

# Out of scope — Live Activity v1

> [!info] Explicitly excluded. Some are natural fast-follows.

- **Persona-flavored waiting.** Copy and accent color keyed to the active Voice + Role. Deferred — persona is already available in the turn, so this is a cheap later add once the base lands.
- **Reasoning / extended-thinking tokens.** Showing the model's private chain-of-thought. Requires substrate work to surface, plus a separate UX/trust debate. v1 narrates *actions*, not private thinking.
- **Redesigning the detailed trace screen.** The existing per-turn audit view (timings, LLM calls, tool details) stays as-is. Live Activity is the glanceable in-chat layer, not a replacement for it.
- **User-facing analytics in the trail.** No per-step timings, token counts, or costs shown in the chat trail.
- **Settings / configurability.** No toggle for animation, verbosity, or trail on/off in v1. One tasteful default.
- **Off-the-record turn narration.** Ephemeral mini-app → assistant turns stay invisible (see [[decisions#D5]]).
- **Tappable/expandable trail steps.** v1 trail is read-only and compact; tapping a step to drill into detail (which could deep-link to the trace screen) is a later idea, not v1.
- **Sound / haptics.** No audio or vibration tied to activity changes.
