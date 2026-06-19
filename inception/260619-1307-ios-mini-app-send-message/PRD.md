---
type: prd
feature: ios-mini-app-send-message
status: draft
created: 2026-06-19
tags:
  - inception/prd
  - feature/ios-mini-app-send-message
  - status/draft
---

# PRD: Mini-apps can ask the assistant on iOS

> [!info] **Status:** Draft / awaiting mob review · **Driver:** SteveCastalk · **Last updated:** 2026-06-19
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items. Follow-up to `260619-1125-ios-mini-app-render`.

## One-line intent

An HTML mini-app running on iOS can ask the assistant a question and get
its answer back, at parity with Android.

## Problem

The prior feature (`ios-mini-app-render`, PR #46) brought HTML mini-apps
to iOS — they render, ask for consent, run scope-gated fetch/store,
persist state, and can be saved. One capability was deliberately left
out: **asking the assistant**. On Android a mini-app can send the
assistant a prompt and receive its reply; on iOS that same call fails
(the host never wired the handler, so the bridge returns a rejected
Promise). A mini-app authored to ask the assistant works on Android and
silently breaks on iOS — the last gap in cross-platform mini-app parity.

## Goals

- [ ] A mini-app on iOS that asks the assistant a question receives the
      assistant's reply, the same as on Android.
- [ ] When no assistant is available yet, the mini-app's request fails
      cleanly (a rejected request it can handle), not a crash — matching
      Android.
- [ ] Android behavior is unchanged.

## Non-goals

- Changing *where* the assistant's reply lands. As on Android today, the
  exchange runs on the user's current conversation and shows up in chat
  history. Isolating it on a throwaway conversation is a separate,
  already-tracked follow-up — promoted to [[out-of-scope]].
- Any new mini-app capability or change to the offerable action set.

## User stories

### Story 1 — A mini-app asks the assistant on iOS

**As an** iOS user running an HTML mini-app, **I want** the mini-app to
be able to ask the assistant and show me the answer, **so that** mini-apps
that rely on the assistant work on my phone just like on Android.

**Acceptance criteria:**
- [ ] When a mini-app on iOS asks the assistant a question, the
      assistant's reply is returned to the mini-app.
- [ ] If the assistant isn't ready yet (e.g. before it has started), the
      mini-app's request fails gracefully and the mini-app keeps working
      — no crash, no hang.
- [ ] The behavior matches Android: the same mini-app asking the same
      question behaves the same way on both platforms.
- [ ] Android's existing ask-the-assistant behavior is unchanged.

## Success metrics

- **Ask-assistant parity** — a mini-app's assistant request succeeds on
  iOS (target: works; today: 0% — always rejected). Verified by the
  iOS-simulator smoke in Story 1.
- **No regression** — Android ask-assistant continues to work; shared
  code compiles and passes on both targets.

## Constraints

- Must not regress Android — the shared change keeps Android's exact
  behavior.
- Verification is compile-both-targets + iOS simulator; real-device smoke
  deferred, consistent with the prior feature ([[decisions]] D3).

## Links

- API contract: `./api-contract.md` (no BE work)
- Context: project-wide `../../CONTEXT.md`
- Issues: `./issues/`
- Predecessor: `../260619-1125-ios-mini-app-render/`
