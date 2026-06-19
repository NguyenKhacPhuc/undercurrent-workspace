---
type: open-questions
feature: ios-mini-app-render
created: 2026-06-19
tags:
  - inception/open-questions
  - feature/ios-mini-app-render
---

# Open questions

> [!success] **No open questions — all resolved with the driver (2026-06-19).**
> The pivotal technical unknown (whether the shared lift needs a substrate
> change) was resolved during Inception scouting: the agent runtime and its
> UI bridge are already shared code with an iOS form, so no `weft` change is
> needed. The Inception open-questions gate is met.

## Open

*(none)*

## Resolved

### Q0 — Does sharing the orchestrator require a substrate (`weft`) change? — 2026-06-19

- **Answer:** No. The orchestrator's only runtime dependency
  (`WeftRuntime` + its `uiBridge`) is defined in shared code with an
  existing iOS form; every other dependency is already shared. The lift
  is a host-only (`undercurrent`) change.
- **By:** Driver (SteveCastalk), from Inception scouting.
- **Promoted to:** [[decisions]] D1, [[PRD]] Problem.

### Q1 — How much Android parity does iOS get in this story? — 2026-06-19

- **Answer:** Full parity (consent + render + scope-gated actions +
  state + save-as) in one story, since the orchestrator lifts atomically
  and a subset is more work, not less.
- **By:** Driver (SteveCastalk).
- **Promoted to:** [[decisions]] D2, [[PRD]] Story 1.
