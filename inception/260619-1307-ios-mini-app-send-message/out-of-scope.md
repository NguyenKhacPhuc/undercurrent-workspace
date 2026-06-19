---
type: out-of-scope
feature: ios-mini-app-send-message
created: 2026-06-19
tags:
  - inception/out-of-scope
  - feature/ios-mini-app-send-message
---

# Out of scope

> [!warning]
> Things this feature is explicitly **not** doing.

- **Ephemeral-conversation isolation for the assistant turn** — the
  exchange runs on the user's current conversation and lands in chat
  history, same as Android. Isolating it is a separate, already-tracked
  follow-up (html-mini-apps Q2 / k04) that should change both platforms
  together. See [[decisions]] D3.
- **Any new mini-app capability or change to the offerable action set** —
  iOS gets exactly the ask-the-assistant behavior Android has today.
- **Android behavior changes** — Android is refactored to use the shared
  factory but its observable behavior is unchanged.
- **Real-device certification** — verification is compile + iOS
  simulator; real-device smoke deferred ([[decisions]] D3 of the
  predecessor feature).
