---
type: issue
feature: ephemeral-assistant-turn
lane: substrate
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 75m
blocked-by: []
tags:
  - inception/issue
  - lane/substrate
  - feature/ephemeral-assistant-turn
  - status/in-progress
  - wave/0
---

# [substrate] The agent can run a one-shot turn that leaves the conversation untouched

**Lane:** substrate (`weft/`)
**PRD section:** [[PRD]] Story 1
**API contract section:** n/a (no BE work)

## Why

Background callers — like a mini-app consulting the assistant — need to
run an assistant turn without disturbing the user's chat. Today every
turn enters the agent's visible history, persists to the conversation
store, and emits the streaming updates the chat surface renders. This
adds a way to run a full turn that does none of that and just returns the
reply.

## Acceptance criteria

The public-facing contract of the new capability (this is a substrate
plumbing slice — stated as observable behavior, in domain terms):

- [ ] The agent can run a single turn that **returns the assistant's
      reply** to the caller.
- [ ] Such a turn leaves the agent's current on-screen conversation
      exactly as it was before — nothing the chat surface renders changes
      because of it.
- [ ] Such a turn does not create or append to any saved conversation.
- [ ] During such a turn, the assistant can still use its tools.
- [ ] A normal turn is unaffected — it still appears in history, persists,
      and streams exactly as before.
- [ ] If the turn produces no assistant reply, the caller is told (so it
      can surface a failure) rather than receiving an empty success.

## Blocked by

- nothing — independently grabbable. (First of the two stories;
  [[02-miniapp-assistant-stays-out-of-chat]] consumes it.)

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Read
> `weft/CLAUDE.md` and open the actual files before editing.

- **Where the turn logic lives:** `weft/harness/agents/.../WeftAgent.kt` —
  the normal turn appends user+assistant entries to the in-memory history,
  syncs them to state, and (when a conversation store is present) appends
  to it. The new capability should run the same model turn but skip the
  history-append + state-sync + conversation-store-append, and avoid
  emitting the streaming/state updates the chat UI observes — returning
  the reply directly.
- **The chat surface follows the agent's state**, so suppressing the
  state emission for this turn is what keeps it off-screen (confirmed by
  scout: `undercurrent` chat renders the shared agent's `state.history`).
- **Naming/shape is Construction's call** — a dedicated suspend entry
  point that returns the reply is likely cleaner than overloading the
  existing reactive `dispatch(Send)` path (which has no direct return).
- **Watch out for:** keep the normal-turn path byte-for-byte behavior;
  add tests that a normal turn still records + persists while the isolated
  turn does neither. Follow the substrate module's existing test
  conventions (`weft/CLAUDE.md`).

## Out of scope for this story

- Suppressing the assistant's memory / tool side-effects (the turn runs
  with normal tools + memory — [[decisions]] D2).
- Streaming the reply (one-shot, returns the final text).
- Any host/mini-app wiring (that's [[02-miniapp-assistant-stays-out-of-chat]]).
