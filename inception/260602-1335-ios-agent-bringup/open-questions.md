---
type: open-questions
feature: ios-agent-bringup
created: 2026-06-02
tags:
  - inception/open-questions
  - feature/ios-agent-bringup
---

# Open questions

> [!question]
> Driver's parking lot for the mob.

## Open

### Q1 — Is the `refactor/extract-auth-strings-to-resources` branch settled enough to build on `main`?

- **Why it matters:** At Inception the `undercurrent` repo had uncommitted changes on that branch (`androidApp/build.gradle.kts`, `AppPreamble.kt`). The iOS DI stories touch `composeApp/iosMain` + `androidApp` DI; if the refactor branch is mid-flight against the same areas, merges will collide. Construction branches from `origin/main` regardless ([[decisions]] D4) — but the mob should confirm the refactor lands (or is abandoned) before the iOS app-wiring stories ([[06-ios-adopt-shared-layer]], [[07-ios-agent-lifecycle]]).
- **[DRIVER GUESS]:** The refactor is a small string-extraction; it merges to `main` soon and doesn't touch the iOS agent path. Proceed, branching from `main`.
- **[ASKED OF]:** All

### Q2 — Does relaxing "feature modules must not depend on Weft" for the chat feature's commonMain need a broader policy note?

- **Why it matters:** `undercurrent/CLAUDE.md` currently forbids feature modules depending on Weft (it broke iOS when Weft was Android-only). The lift ([[decisions]] D2) makes the chat feature's commonMain depend on the now-KMP substrate. That's safe, but the rule's rationale is now obsolete for KMP-published substrate modules.
- **[DRIVER GUESS]:** Update the CLAUDE.md note to "feature modules may depend on KMP-published substrate modules from commonMain; still avoid Android-only substrate bits." Done as part of [[05-share-chat-streaming]].
- **[ASKED OF]:** Android / iOS

## Resolved

### Q0 — MVP boundary? — 2026-06-02

- **Answer:** Working text agent + OAuth integrations (stories 1–6 + OAuth of the scoping map). Koog-bits + voice deferred.
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D1, [[out-of-scope]]

### Q0b — iOS-only copies or lift to commonMain? — 2026-06-02

- **Answer:** Lift the Weft-backed layer to commonMain (shared).
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D2
