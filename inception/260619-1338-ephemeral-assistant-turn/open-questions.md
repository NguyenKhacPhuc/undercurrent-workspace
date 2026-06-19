---
type: open-questions
feature: ephemeral-assistant-turn
created: 2026-06-19
tags:
  - inception/open-questions
  - feature/ephemeral-assistant-turn
---

# Open questions

> [!success] **No open questions (2026-06-19).**
> The architecture was scouted during Inception and the isolation approach
> chosen by the driver (substrate one-shot turn — [[decisions]] D1). Scope
> is deliberately bounded to conversation-transcript + saved-history
> isolation ([[decisions]] D2); memory/tool isolation is an explicit
> non-goal. The Inception open-questions gate is met.

## Open

*(none)*

## Resolved

### Q0 — Which isolation mechanism? — 2026-06-19

- **Answer:** A substrate one-shot ("headless") turn on the live agent
  that returns the reply but doesn't enter the agent's visible history,
  doesn't persist to the conversation store, and doesn't emit the
  streaming updates the chat UI observes. Chosen over a host-only
  throwaway agent (rebuild latency, not reusable) and an `ephemeral` send
  flag (more invasive, separate reply path).
- **By:** Driver (SteveCastalk), from an architecture scout.
- **Promoted to:** [[decisions]] D1.

### Q1 — Does "isolated" include the assistant's memory / tool effects? — 2026-06-19

- **Answer:** No — v1 isolates the conversation transcript + saved
  history only; the turn otherwise behaves normally (tools + memory work).
  Memory-write isolation is a deferred follow-up.
- **By:** Driver (SteveCastalk).
- **Promoted to:** [[decisions]] D2, [[out-of-scope]].
