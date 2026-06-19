---
type: decisions
feature: ios-mini-app-send-message
created: 2026-06-19
tags:
  - inception/decisions
  - feature/ios-mini-app-send-message
---

# Decisions

> [!info]
> ADR-lite log. Driver-made calls for the mob to ratify.

### D1 — Share the assistant-handler factory, don't write an iOS twin — 2026-06-19

- **Context:** Android's ask-the-assistant handler is a private factory
  in the Android app's DI module; its logic (run a one-shot agent turn,
  return the reply) uses only shared types. iOS needs the same behavior.
- **Options considered:** (a) duplicate the handler on iOS; (b) lift the
  factory to shared code and have both platforms supply it.
- **Decision:** Lift it to shared code; both platforms use one factory.
- **Why:** Single source of truth, no behavior drift, less code —
  consistent with how the orchestrator was shared in the predecessor
  feature.
- **Consequences:** Both platforms commit to identical
  ask-the-assistant behavior; a future platform difference would need an
  explicit seam.

### D2 — Drive the handler off the current agent, not a per-platform AgentSession — 2026-06-19

- **Context:** Android reaches the live agent through an `AgentSession`
  it builds inside its app view-model; iOS doesn't construct one. The
  handler only needs the *current agent* to run a turn, which both
  platforms already expose through the shared agent slot.
- **Decision:** Parameterize the shared factory on "the current agent"
  (supplied from the shared agent slot), so iOS needs no new
  AgentSession wiring.
- **Why:** Narrowest dependency, mirrors how the predecessor feature
  narrowed its dependency to what it actually used; avoids standing up
  iOS infrastructure the handler doesn't need.
- **Consequences:** If a future caller needs more than the current agent,
  the factory's input would widen then.

### D3 — Keep Android's "runs on the current conversation" behavior; defer isolation — 2026-06-19

- **Context:** Android's assistant turn runs on the user's current
  conversation, so it lands in chat history. This is a known v1 caveat
  already tracked (html-mini-apps Q2 / k04).
- **Decision:** iOS matches Android exactly — no ephemeral-conversation
  isolation in this feature.
- **Why:** Parity is the goal here; isolation is a separate concern that
  should change both platforms together when it's tackled.
- **Consequences:** A mini-app's assistant exchange remains visible in
  chat history on both platforms until the isolation follow-up ships.
