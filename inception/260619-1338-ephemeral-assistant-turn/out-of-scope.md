---
type: out-of-scope
feature: ephemeral-assistant-turn
created: 2026-06-19
tags:
  - inception/out-of-scope
  - feature/ephemeral-assistant-turn
---

# Out of scope

> [!warning]
> Things this feature is explicitly **not** doing.

- **Isolating the assistant's memory / tool side-effects.** The isolated
  turn runs with the assistant's normal tools and memory; only the chat
  transcript and saved conversation are kept clean. Barring a mini-app
  query from updating durable memory is a separate future tightening
  ([[decisions]] D2).
- **Streaming the reply into the mini-app.** The mini-app gets the final
  answer (non-streaming), same as today.
- **A visible "a mini-app asked the assistant" indicator or audit trail.**
  The exchange is silent; surfacing it is a separate product decision.
- **Changing normal chat-turn behavior.** Regular turns still appear and
  persist exactly as before.
- **Real-device certification.** Verification is compile + tests +
  simulator/on-device smoke; real-device deferred, consistent with the
  prior mobile features.
