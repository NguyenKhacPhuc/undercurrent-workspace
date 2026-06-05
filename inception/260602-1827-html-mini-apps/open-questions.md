---
type: open-questions
feature: html-mini-apps
created: 2026-06-02
tags:
  - inception/open-questions
  - feature/html-mini-apps
---

# Open questions

> [!question]
> Driver's parking lot for the mob. The big calls (escape-hatch framing, per-app-scope security, full surface, end-to-end scope) are in [[decisions]]. These need the mob.

## Open

### Q1 — Which actions are *offerable* to mini-apps, and do destructive ones need extra friction?

- **Why it matters:** The user approves from a menu of "offerable" actions ([[decisions]] D2). The host defines that menu. Read-mostly actions (fetch, store, share, clipboard-read) are clearly fine to offer. Destructive/sensitive ones (delete data, send a message, spend money, location) may warrant extra friction (a per-use confirm, not just a one-time grant) or exclusion in v1.
- **[DRIVER GUESS]:** v1 offers a **read-mostly** set (allowlisted http, key-value store, share, clipboard-read, system info). Destructive/sensitive actions are **excluded from the offerable menu in v1**; revisit per-action with a stronger consent model later.
- **[ASKED OF]:** Product / All — see [[02-offerable-actions]].

### Q4 — Should native-component (Tier-A) mini-apps persist interaction state for revisit, like HTML mini-apps do?

- **Why it matters:** Story 04 gave HTML mini-apps `window.weft.getState/setState` over a `MiniAppStateStore`. Native-component mini-apps (the `ui_render` tree path) currently restore their cached *layout* on revisit but not the user's *interaction* state (field values, toggles). Raised on weft PR #21.
- **[DRIVER GUESS]:** Yes, and reuse the same store — `MiniAppStateStore` is deliberately render-path-agnostic (keyed by `miniAppId`, opaque JSON, nothing HTML-specific). The missing piece is a **binding** on the native render path: the substrate's `TreeRenderer` (`:compose`) would need a seam to read/write the store for the components whose values count as persistable state. Spin as its own substrate story (reuses the store; the work is the native-side binding) rather than expanding the HTML feature.
- **[ASKED OF]:** Substrate / Product.

## Resolved

### Q0 — Escape hatch or replacement? — 2026-06-02
- **Answer:** Escape hatch; native palette stays the default. → [[decisions]] D1.

### Q0b — Security model? — 2026-06-02
- **Answer:** Per-mini-app declared scopes + user approval. → [[decisions]] D2.

### Q0c — Bridge surface depth + feature scope? — 2026-06-02
- **Answer:** Full `window.weft` surface; substrate bridge + host catalog (end-to-end). → [[decisions]] D3, D4.

### Q2 — `sendMessage` foreground vs silent? — 2026-06-05
- **Answer:** **Silent** — the result returns to the mini-app, no chat navigation. Built in [[04-mini-app-asks-assistant]] (#29). **v1 caveat:** the turn currently runs on the user's *current* conversation, so it lands in chat history (the resolved intent was "not in the visible thread"); isolating it on an ephemeral conversation is a tracked follow-up.

### Q3 — Agent guidance: HTML vs native component? — 2026-06-05
- **Answer:** Added to the app preamble — prefer native components for standard UI; reach for an HTML mini-app only for bespoke client-side logic / novel interaction; save for reuse via `create_html_mini_app` when the user asks. Shipped with [[07-create-html-mini-app]] (#30). Tune against real behavior.
