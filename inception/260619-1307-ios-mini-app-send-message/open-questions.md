---
type: open-questions
feature: ios-mini-app-send-message
created: 2026-06-19
tags:
  - inception/open-questions
  - feature/ios-mini-app-send-message
---

# Open questions

> [!success] **No open questions (2026-06-19).**
> This is a parity gap with an established Android precedent and no open
> product decisions — the handler mirrors Android exactly, including the
> "no agent ready → rejected request" behavior and the
> conversation-history caveat (out of scope). The technical path was
> confirmed during Inception scouting (the current agent is reachable via
> the shared agent slot on both platforms; no substrate change). The
> Inception open-questions gate is met.

## Open

*(none)*

## Resolved

### Q0 — Does iOS need new agent infrastructure to support this? — 2026-06-19

- **Answer:** No. The shared agent slot already exposes the current
  agent on iOS; the handler is driven off that, so no `AgentSession`
  wiring or substrate change is needed.
- **By:** Driver (SteveCastalk), from Inception scouting.
- **Promoted to:** [[decisions]] D2.
