---
type: decisions
feature: ephemeral-assistant-turn
created: 2026-06-19
tags:
  - inception/decisions
  - feature/ephemeral-assistant-turn
---

# Decisions

> [!info]
> ADR-lite log. Driver-made calls for the mob to ratify.

### D1 — Isolate via a substrate one-shot ("headless") turn capability, not a host workaround — 2026-06-19

- **Context:** The chat surface renders the single shared agent's
  in-memory history, and turns also persist to the conversation store —
  so a mini-app turn pollutes both. Options were: (a) a substrate
  capability to run a turn that touches neither; (b) a host-only
  throwaway agent per call; (c) an `ephemeral` flag on the send intent.
- **Decision:** Add the capability to the substrate (`weft`) — run a
  full assistant turn that returns its reply without entering the agent's
  visible history or the conversation store, and without emitting the
  streaming updates the chat UI observes.
- **Why:** Reuses the live agent (no per-call rebuild, so low latency;
  tools and persona come for free), keeps semantics clean, and is
  reusable by any Weft host — consistent with undercurrent's rule that
  the SDK provides capabilities and the app just registers. The host-only
  agent (b) adds rebuild latency and credential plumbing and helps no
  other host; the intent flag (c) is more invasive to the reactive
  surface and needs a separate reply path.
- **Consequences:** A small, well-scoped addition to the substrate agent
  API. The host change becomes a one-liner. Cross-repo ordering applies
  (D3).

### D2 — Isolate the conversation only; tools and memory behave as a normal turn — 2026-06-19

- **Context:** "Isolation" could mean just the visible transcript, or
  also the assistant's memory/tool side-effects.
- **Decision:** v1 isolates the **conversation transcript + saved
  history** only. The turn otherwise behaves like any normal turn —
  the assistant uses its tools and memory as usual.
- **Why:** That's the actual reported problem (stray Q&A in the chat /
  history). Suppressing memory/tool effects is a different, larger
  concern and would diverge the turn from normal behavior; defer it until
  there's evidence it's needed.
- **Consequences:** A mini-app's assistant query can still, e.g., trigger
  a memory write if the assistant chooses to — same as a normal question.
  Tightening this is a tracked follow-up ([[out-of-scope]]).

### D3 — Substrate PR merges first; the host PR consumes it — 2026-06-19

- **Context:** The feature spans `weft/` (the capability) and
  `undercurrent/` (the mini-app wiring). `weft` is a composite build
  (`includeBuild("../weft")`), so the host branch can build against the
  local weft branch during development.
- **Decision:** Land the substrate story first; the host story is
  `blocked-by` it and merges after.
- **Why:** Avoids merging a host change that references a substrate API
  not yet on weft's main.
- **Consequences:** Two PRs in two repos, sequenced. The host story
  starts once the substrate API shape is settled (can develop against the
  weft branch before it merges).
