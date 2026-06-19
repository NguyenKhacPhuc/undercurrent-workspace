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
> Driver's parking lot for the mob. The big calls (escape-hatch framing, per-app-scope security, full surface, end-to-end scope) are in [[decisions]]. All questions now resolved.

## Open

*(none)*

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

### Q1 — Which actions are offerable to mini-apps? — 2026-06-19
- **Answer:** v1 offers a **read-mostly** set (allowlisted http, key-value store, share, clipboard-read, system info). Destructive/sensitive actions are **excluded from the offerable menu in v1**; revisit per-action with a stronger consent model later. See [[02-offerable-actions]].

### Q4 — Native-component (Tier-A) mini-app state persistence? — 2026-06-19
- **Answer:** Yes — reuse `MiniAppStateStore` (render-path-agnostic); spun as a future substrate story (native render-path binding) rather than expanding the HTML feature.
