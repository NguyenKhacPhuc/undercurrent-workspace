---
type: prd
feature: ios-mini-app-render
status: draft
created: 2026-06-19
tags:
  - inception/prd
  - feature/ios-mini-app-render
  - status/draft
---

# PRD: HTML mini-apps render on iOS

> [!info] **Status:** Draft / awaiting mob review · **Driver:** SteveCastalk · **Last updated:** 2026-06-19
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

HTML mini-apps render and work fully on iOS, at parity with Android.

## Problem

iOS already lets a user *create and save* an HTML mini-app (the agent
authors it, it lands in the catalog). But **tapping a saved HTML
mini-app on iOS does nothing useful** — instead of rendering the
mini-app, the host just replays its trigger prompt through the chat.
First-run consent, the instant HTML render, saved-state, and
save-as-mini-app are all silently no-ops on iOS.

The renderer itself is not the problem: the substrate's iOS WebView
component is fully built and the host's bridge bindings are already
wired. The gap is purely in the **host orchestration layer**, which
was written during the Android-primary phase and never shared with
iOS. The result is that the marquee mini-apps feature is
Android-only in practice, even though iOS users can produce mini-apps
they then can't open.

## Goals

Testable goals.

- [ ] Tapping a saved HTML mini-app on iOS opens and renders it (its
      HTML content paints on screen), rather than replaying a chat turn.
- [ ] An HTML mini-app on iOS reaches the same behavior set as Android:
      first-run consent, scope-gated host actions, saved state, and
      save-as-mini-app.
- [ ] Android mini-app behavior is unchanged (zero regression).

## Non-goals

- Native-component (non-HTML) mini-apps gaining new interaction-state
  persistence — promoted to [[out-of-scope]] (a separate future
  substrate story).
- Any new host action, capability, or change to the offerable set.
- On-device hardware certification — see [[decisions]] D3.

## User stories

### Story 1 — Saved HTML mini-apps work on iOS

**As an** iOS user, **I want** the HTML mini-apps I save to open and
run when I tap them, **so that** the mini-apps feature works the same
on my phone as it does on Android.

**Acceptance criteria:**
- [ ] Tapping a saved HTML mini-app on iOS opens it and renders its HTML
      content on screen (not a chat reply).
- [ ] The first time an HTML mini-app with declared capabilities runs on
      iOS, the user is asked to approve those capabilities before
      anything renders; approving lets the mini-app run with exactly
      those capabilities, denying lets it run with none.
- [ ] An HTML mini-app that declares no capabilities renders directly on
      iOS with no consent prompt.
- [ ] A rendered HTML mini-app on iOS can use its approved host actions
      (e.g. fetch, stored data) and is refused any action it was not
      granted. *(The `sendMessage`/ask-assistant binding is deferred — see
      [[out-of-scope]]; it needs iOS chat-DI wiring beyond this lift.)*
- [ ] A mini-app's saved data persists on iOS — closing and reopening it
      restores what it stored.
- [ ] An iOS user can save the current on-screen result as a reusable
      mini-app, and reopening it paints instantly without re-running the
      agent.
- [ ] Revisiting a previously-opened mini-app on iOS restores its last
      rendered view.
- [ ] Android mini-app behavior (open, consent, actions, state, save) is
      unchanged.

## Success metrics

- **Mini-app open success on iOS** — tapping a saved HTML mini-app
  renders it (target: works; today: 0% — never renders). Measured by
  the iOS simulator verification in Story 1's acceptance.
- **Behavior parity** — the four mini-app behaviors (consent, render,
  state, save-as) all function on iOS, matching Android. Measured by
  shared-code coverage compiling and passing on both targets.

## Constraints

- Must not regress Android — the shared lift keeps the exact behavior
  the Android orchestrator has today.
- Verification is compile-both-targets + iOS simulator launch;
  on-device smoke is deferred ([[decisions]] D3), consistent with prior
  mobile features in this workspace.

## Links

- API contract: `./api-contract.md` (no BE work)
- Context: project-wide `../../CONTEXT.md`
- Issues: `./issues/`
