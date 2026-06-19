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

*(none)*

## Resolved

### Q0 — MVP boundary? — 2026-06-02

- **Answer:** Working text agent + OAuth integrations (stories 1–6 + OAuth of the scoping map). Koog-bits + voice deferred.
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D1, [[out-of-scope]]

### Q0b — iOS-only copies or lift to commonMain? — 2026-06-02

- **Answer:** Lift the Weft-backed layer to commonMain (shared).
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D2

### Q1 — Is the `refactor/extract-auth-strings-to-resources` branch settled enough to build on `main`? — 2026-06-19

- **Answer:** The refactor is a small string-extraction; it merges to `main` soon and doesn't touch the iOS agent path. Proceed, branching from `main`.
- **By:** Driver (SteveCastalk)

### Q2 — Does relaxing "feature modules must not depend on Weft" for the chat feature's commonMain need a broader policy note? — 2026-06-19

- **Answer:** Update the CLAUDE.md note to "feature modules may depend on KMP-published substrate modules from commonMain; still avoid Android-only substrate bits." Done as part of [[05-share-chat-streaming]].
- **By:** Driver (SteveCastalk)
