---
type: feature-index
feature: html-mini-apps
status: draft
created: 2026-06-02
tags:
  - inception/index
  - feature/html-mini-apps
  - status/draft
---

# HTML mini-apps with an agent bridge — feature index

> [!info] **Status:** Draft / awaiting mob review
> Give the agent's flexible HTML surface a permissioned JS↔agent bridge so mini-apps can *do* things — the Tier-B escape hatch alongside the native component palette. Full bridge surface, per-mini-app declared scopes + approval, substrate bridge + host catalog ([[decisions]]).

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] (no BE work)
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

12 stories across two lanes: **`substrate`** (`weft/` — the `window.weft` bridge on the existing `HtmlComponent`) and **`kmp-common`** (`undercurrent/` — the HTML-doc mini-app catalog, the offerable-action policy, and the consent UX). Host stories consume the substrate via the bumped `weft` submodule.

> [!tip] Wave = 0 if `blocked-by` is empty, else `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-bridge-call-action]] | 75m | substrate |
| [[02-mini-app-theme]] | 45m | substrate |

### 🟡 Wave 1 — unlocked once the bridge foundation lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[03-scope-gate]] | 75m | substrate | `s01` |
| [[04-mini-app-state]] | 60m | substrate | `s01` |
| [[05-live-updates]] | 60m | substrate | `s01` |
| [[06-mini-app-lifecycle]] | 60m | substrate | `s01` |
| [[07-ask-assistant-hook]] | 60m | substrate | `s01` |

### 🟠 Wave 2 — sealing + the host catalog

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[08-sandbox-hardening]] | 60m | substrate | `s01`, `s03` |
| [[01-html-mini-apps]] | 75m | kmp-common | `s01`, `s03` |
| [[02-offerable-actions]] | 45m | kmp-common | `s03` |

### 🔵 Wave 3 — consent + agent power

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[03-approve-on-first-run]] | 75m | kmp-common | `k01`, `k02`, `s03` |
| [[04-mini-app-asks-assistant]] | 60m | kmp-common | `s07`, `k01` |

### ⚪ Wave 4 — follow-ups (surfaced during `k01` build, PR #25)

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[05-tighten-miniapp-http-allowlist]] | 45m | kmp-common | — (builds on `k01`) |
| [[09-invoker-mini-app-id]] | 60m | substrate | — (unlocks host `store_*`) |

> [!note] **Cross-repo gate.** Substrate (`weft/`) stories land as PRs in `weft/` and must be **bumped into the workspace `weft` submodule** before the `kmp-common` (`undercurrent/`) stories that consume them are grabbable. The bridge foundation ([[01-bridge-call-action]] + [[03-scope-gate]]) is the critical cross-repo dependency.

---

## Status snapshot (2026-06-05)

**Done (on `main`):** all substrate s01–s09 · kmp-common k01, k02, k03, **k06 render-on-tap** · host-consume of s09 (`store_*` live). A tapped HTML mini-app renders instantly through the bridged `Html` component, scope-gated, asks first-run consent, and can `http_fetch` + `store_*`. Android-primary (iOS render still stubbed).

**Remaining — each needs a focused session:**
- [[04-mini-app-asks-assistant]] — s07's `sendMessage` bridge is ready; wiring a **silent agent turn** (decided) needs a host path to the agent (the one-shot `WeftAgent.send` is `internal`; needs collecting a streamed reply, likely an ephemeral conversation).
- [[05-tighten-miniapp-http-allowlist]] — decided: default allowlist + prefs-backed additions. Needs a small allowlist repo + a default-host product call + (ideally) a settings surface to add hosts.

---

## All issues

```
issues/
├── substrate/   (weft/ — the window.weft bridge)
│   ├── 01-bridge-call-action.md      (W0)
│   ├── 02-mini-app-theme.md          (W0)
│   ├── 03-scope-gate.md              (W1 ← s01)
│   ├── 04-mini-app-state.md          (W1 ← s01)
│   ├── 05-live-updates.md            (W1 ← s01)
│   ├── 06-mini-app-lifecycle.md      (W1 ← s01)
│   ├── 07-ask-assistant-hook.md      (W1 ← s01)
│   ├── 08-sandbox-hardening.md       (W2 ← s01,s03)
│   └── 09-invoker-mini-app-id.md     (W4 follow-up)
└── kmp-common/  (undercurrent/ — catalog + policy + consent)
    ├── 01-html-mini-apps.md          (W2 ← s01,s03)  [done]
    ├── 02-offerable-actions.md       (W2 ← s03)      [done]
    ├── 03-approve-on-first-run.md    (W3 ← k01,k02,s03) [done]
    ├── 04-mini-app-asks-assistant.md (W3 ← s07,k01)  [remaining]
    ├── 06-render-html-mini-app.md    (W3 foundation)  [done]
    └── 05-tighten-miniapp-http-allowlist.md (W4 follow-up)
```

> [!note] **Updating this index** — flip each issue's `status` (`ready` → `in-progress` → `done`). Substrate PRs land in `weft/`; host PRs in `undercurrent/`.

---

## Definition of done (whole feature)

- [ ] All issues `done`.
- [ ] [[open-questions]] resolved (Q1 offerable set, Q2 sendMessage surfacing, Q3 agent guidance).
- [ ] [[PRD]] has ≥1 success metric. ✅ (four)
- [ ] [[api-contract]] has zero `TBD`. ✅ (no BE)
- [ ] A demo HTML mini-app calls an approved action + persists state + adopts the theme, end-to-end on a device; an un-approved call is provably refused; both targets compile with no Android regression.
