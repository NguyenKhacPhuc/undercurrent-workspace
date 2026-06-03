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

### Q2 — Does `sendMessage` (a mini-app firing an agent turn) run in the foreground chat, or silently?

- **Why it matters:** A mini-app calling "ask the assistant" could (a) open/append to the visible chat thread, or (b) run a hidden turn whose result returns only to the mini-app. (a) is transparent but disruptive; (b) is seamless but hides agent activity from the user.
- **[DRIVER GUESS]:** **Result returns to the mini-app** (b-style), but the turn is **recorded in traces** so it's auditable; not injected into the visible chat thread. Revisit if it feels opaque.
- **[ASKED OF]:** Product / iOS / Android — see [[04-mini-app-asks-assistant]].

### Q3 — How should the agent decide HTML vs a native component?

- **Why it matters:** With two rendering paths ([[decisions]] D1), the agent needs guidance or it will over/under-use HTML. This is a prompt/docs concern, not code — but it determines whether the feature actually helps.
- **[DRIVER GUESS]:** Add a short rule to the app preamble: *prefer a native component for structured/standard UI; reach for an HTML mini-app only for custom logic / novel interaction / bespoke viz the palette can't express.* Tune against real behavior.
- **[ASKED OF]:** All.

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
