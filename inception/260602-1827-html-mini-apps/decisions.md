---
type: decisions
feature: html-mini-apps
created: 2026-06-02
tags:
  - inception/decisions
  - feature/html-mini-apps
---

# Decisions

> [!info]
> ADR-lite log. Driver decisions for the mob to ratify.

---

### D1 — HTML+bridge is an escape hatch, not a replacement for the native palette — 2026-06-02

- **Context:** The agent renders mini-app UI from 147 native components — excellent for structured/standard UI, but it can't express novel shapes without an engineer shipping a new component. The substrate's `HtmlComponent` already renders arbitrary HTML/JS but is sandboxed (no bridge).
- **Options considered:** (a) keep extending the native palette; (b) replace it with HTML; (c) keep native as the default + add HTML+bridge for the flexible long tail.
- **Decision:** **(c).** Native palette stays the default for Tier-A (structured, standard, themed) UI; HTML+bridge becomes the Tier-B path for custom logic / novel interaction / bespoke viz. A thin Tier-C (live camera/map/AR/heavy real-time) stays native-only and is out of scope.
- **Why:** 147 hand-built components don't scale — every novel mini-app needs an engineer. HTML+bridge lets the agent author the long tail itself, while native stays better for the common, structured cases.
- **Consequences:** Two rendering paths to maintain. Guidance needed (preamble/docs) on when the agent should reach for HTML vs a native component.

### D2 — Security: per-mini-app declared scopes + user approval — 2026-06-02

- **Context:** Mini-app JS is agent-authored and semi-untrusted (prompt-injection risk). The bridge is the boundary between untrusted JS and real device/app actions.
- **Options considered:** conservative default allowlist + host extend; **per-mini-app declared scopes + user approval**; agent-mediated only (no direct tool calls).
- **Decision:** **Per-mini-app declared scopes + approval.** Each mini-app declares the actions it needs; the user approves/denies on first run (app-permission style); the substrate enforces the approved set at the bridge boundary; the host stores the grants and surfaces the consent UX. Network only via the existing allowlist; CSP + WebView sandbox enforced.
- **Why:** Strongest user control, and the consent UX makes the capability legible — the user always knows what a flexible mini-app can touch. Worth the extra consent-UX + grant-storage cost given untrusted JS.
- **Consequences:** Adds a consent screen + a per-mini-app grant store + a scope-enforcement gate. The host must define the **offerable** action menu (which actions are ever approvable). Destructive/sensitive actions may warrant extra friction (flagged in [[open-questions]]).

### D3 — Full bridge surface in v1 — 2026-06-02

- **Context:** The `window.weft` API could ship minimal (callTool only) or full.
- **Decision:** **Full:** `callTool` (invoke an approved action) + `getState`/`setState` (per-mini-app persistence) + theme tokens (CSS vars) + `sendMessage` (fire an agent turn) + `onData` (native→JS live push) + lifecycle (open/close) + multi-instance.
- **Why:** The driver wants a genuinely usable mini-app runtime, not a proof of concept.
- **Consequences:** Larger surface → more stories. `onData` / lifecycle / multi-instance are the most speculative before real usage; sliced so they can be deprioritized if needed (see [[_index]] waves).

### D4 — Scope: substrate bridge + host catalog (end-to-end) — 2026-06-02

- **Context:** The feature could stop at the substrate bridge mechanism, or include the host change that makes a mini-app *be* an HTML doc.
- **Decision:** **Both.** The substrate ships the bridge; the host (`undercurrent/`) makes a mini-app saveable as an HTML doc (+ declared scopes + state), cached for render-on-tap, with the approval UX and the agent-turn wiring.
- **Why:** Delivers an actually-usable HTML mini-app end-to-end, not just a mechanism with no consumer.
- **Consequences:** Cross-repo feature (weft + undercurrent). Host stories depend on substrate stories landing + being bumped into the workspace `weft` submodule.
