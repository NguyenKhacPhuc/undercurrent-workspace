---
type: feature-index
feature: ephemeral-assistant-turn
status: ready
created: 2026-06-19
tags:
  - inception/index
  - feature/ephemeral-assistant-turn
  - status/ready
---

# A mini-app's assistant question stays out of the chat — feature index

> [!info] **Status:** Draft / awaiting mob review
> Cross-repo, two-story feature (substrate capability → host wiring).
> Closes the last v1 caveat of the mini-app features. Open questions resolved.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] *(no BE work)*
- Decisions: [[decisions]] *(D1–D3)*
- Open questions: [[open-questions]] *(0 open)*
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

> [!note]
> The two stories are **sequential, not parallel** — the host story
> consumes the substrate capability. This is inherent (you can't wire a
> capability that doesn't exist yet), so the waves are single-issue by
> design. The host story can be *developed* against the `weft` branch
> before it merges; it just can't *merge* first ([[decisions]] D3).

### 🟢 Wave 0 — start here

| Issue | Estimate | Lane | Repo |
|---|---|---|---|
| [[01-isolated-one-shot-turn]] | 75m | substrate | `weft/` |

### 🟡 Wave 1 — unlocked once wave 0 lands

| Issue | Estimate | Lane | Repo | Blocked by |
|---|---|---|---|---|
| [[02-miniapp-assistant-stays-out-of-chat]] | 30m | kmp-common | `undercurrent/` | `01` |

---

## All issues

```
issues/
├── substrate/
│   └── 01-isolated-one-shot-turn.md             (W0 — weft)
└── kmp-common/
    └── 02-miniapp-assistant-stays-out-of-chat.md (W1 ← 01 — undercurrent)
```

> [!note] **How to update this index as work lands**
> Flip each issue's `status` (`ready` → `in-progress` → `done`).
> Story 01 is a PR in `weft/`; story 02 is a PR in `undercurrent/`.

---

## Definition of done (whole feature)

- [ ] Both issues have status `done`.
- [x] [[open-questions]] has zero unresolved items. *(2026-06-19.)*
- [x] [[PRD]] has at least one success metric. *(two — clean chat, answer delivered.)*
- [x] [[api-contract]] has zero `TBD` markers. *(No BE work.)*
- [ ] [[decisions]] reviewed by the mob.
- [ ] `weft` + `undercurrent` build green on both targets with the
      relevant tests, no normal-chat regression.
- [ ] On device/simulator: a mini-app asks the assistant, gets an answer,
      and nothing new appears in the chat thread or conversation list —
      on Android and iOS. *(Real-device deferred.)*
