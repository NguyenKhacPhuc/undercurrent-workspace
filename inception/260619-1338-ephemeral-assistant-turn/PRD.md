---
type: prd
feature: ephemeral-assistant-turn
status: draft
created: 2026-06-19
tags:
  - inception/prd
  - feature/ephemeral-assistant-turn
  - status/draft
---

# PRD: A mini-app's question to the assistant stays out of the chat

> [!info] **Status:** Draft / awaiting mob review · **Driver:** SteveCastalk · **Last updated:** 2026-06-19
> See [[_index]] for the parallel-work plan and [[open-questions]]. Cross-platform follow-up to `260619-1307-ios-mini-app-send-message`.

## One-line intent

When a mini-app asks the assistant a question, the exchange is answered
but does not appear in the user's chat thread or conversation list.

## Problem

A mini-app can ask the assistant (`window.weft.sendMessage`) on both
platforms now. But the turn runs on the user's **current conversation**:
the question and answer append to the live chat transcript and the saved
conversation. So a weather widget quietly asking "what's the forecast?"
dumps a stray Q&A into the middle of the user's actual chat and leaves it
in their history. The mini-app should be able to consult the assistant
without hijacking or polluting the user's conversation. This is the known
v1 caveat carried by both prior mini-app features.

## Goals

- [ ] A mini-app's assistant question is answered (the mini-app gets the
      reply) **without** the question or answer appearing in the visible
      chat thread.
- [ ] The exchange does **not** create or modify any entry in the user's
      conversation list / saved history.
- [ ] The assistant answers with its normal capabilities (it can use its
      tools) and as its normal self (same persona/context).
- [ ] Behavior is identical on Android and iOS.

## Non-goals

- **Isolating the assistant's memory or tool side-effects.** The turn runs
  with the assistant's normal tools and memory, exactly like any other
  turn — only the *conversation transcript and saved history* are kept
  clean. If a mini-app query should also be barred from updating durable
  memory, that's a separate future tightening — promoted to [[out-of-scope]].
- **Streaming the reply into the mini-app.** The mini-app gets the final
  answer, as today (non-streaming).
- **A user-visible "mini-app asked the assistant" indicator.** Out of
  scope; the exchange is simply silent.

## User stories

### Story 1 — The SDK can run an assistant turn that leaves the conversation untouched (substrate)

**As a** Weft host, **I want** to run a one-shot assistant turn whose
question and answer never enter the visible conversation or its saved
history, **so that** background callers (like a mini-app) can consult the
assistant without disturbing the user's chat.

**Acceptance criteria:**
- [ ] A turn run this way returns the assistant's reply to the caller.
- [ ] The turn leaves the agent's on-screen conversation exactly as it was
      — nothing is added to what the chat surface shows.
- [ ] The turn does not create or append to any saved conversation.
- [ ] The assistant can still use its tools during the turn.
- [ ] A normal (non-isolated) turn is completely unaffected — it still
      appears and is saved as before.

### Story 2 — A mini-app's assistant question stays out of the user's chat (host)

**As a** user running a mini-app, **I want** the mini-app's questions to
the assistant to be answered without showing up in my chat, **so that** my
conversation stays mine and isn't cluttered by what mini-apps ask.

**Acceptance criteria:**
- [ ] After a mini-app asks the assistant, the question and answer do not
      appear in the chat thread on screen.
- [ ] The exchange does not appear in the conversation list / history.
- [ ] The mini-app still receives the assistant's answer and continues
      working.
- [ ] If the assistant isn't ready, the mini-app's request still fails
      gracefully (unchanged from today).
- [ ] Works identically on Android and iOS.

## Success metrics

- **Clean chat** — after a mini-app assistant call, zero new entries in
  the visible thread or conversation list (target: 0; today: 2 — the Q
  and the A). Verified by the story's on-device/simulator smoke.
- **Answer still delivered** — the mini-app receives a non-empty reply,
  same as today.

## Constraints

- Must not change normal chat-turn behavior on either platform.
- Substrate change lands in `weft/` first; the host change consumes it
  ([[decisions]] D3).
- Verification: both targets compile + the relevant tests; on-device /
  simulator smoke (real-device deferred, consistent with prior features).

## Links

- API contract: `./api-contract.md` (no BE work)
- Context: project-wide `../../CONTEXT.md`
- Issues: `./issues/`
- Predecessors: `../260619-1307-ios-mini-app-send-message/`, `../260602-1827-html-mini-apps/`
