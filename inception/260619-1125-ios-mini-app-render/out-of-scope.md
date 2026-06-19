---
type: out-of-scope
feature: ios-mini-app-render
created: 2026-06-19
tags:
  - inception/out-of-scope
  - feature/ios-mini-app-render
---

# Out of scope

> [!warning]
> Things this feature is explicitly **not** doing.

- **`window.weft.sendMessage` (ask-the-assistant) binding on iOS** —
  discovered during Construction: Android wires a
  `MiniAppAssistantHandler` into its component registry via an
  `AgentSession`, but iOS DI exposes no equivalent agent provider, so
  this needs separate chat-DI wiring beyond the orchestrator lift.
  Deferred to a small follow-up story. The orchestrator parity (render +
  consent + fetch/store + state + save-as) ships here. A mini-app
  calling `sendMessage` on iOS gets a rejected Promise, not a crash.

- **Native-component (non-HTML) mini-app interaction-state persistence** —
  restoring field values / toggles for the `ui_render`-tree path is a
  separate future substrate story (tracked as html-mini-apps Q4). This
  feature only brings the existing **HTML** mini-app behavior to iOS.
- **Any new host action or change to the offerable capability set** — iOS
  gets exactly the actions Android offers today, no more.
- **Ephemeral-conversation isolation for `sendMessage`** — the known v1
  limitation that a mini-app's assistant turn lands in the visible chat
  thread is a separate tracked follow-up from the html-mini-apps feature,
  unchanged here.
- **On-device hardware certification** — deferred per [[decisions]] D3;
  verification is compile + iOS simulator.
- **New mini-app authoring/creation UX on iOS** — creation already works;
  this feature is about *opening and running* saved mini-apps.
