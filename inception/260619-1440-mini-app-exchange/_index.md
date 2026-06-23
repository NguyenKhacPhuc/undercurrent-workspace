---
type: feature-index
feature: mini-app-exchange
status: draft
created: 2026-06-19
tags:
  - inception/index
  - feature/mini-app-exchange
  - status/draft
---

# Mini-App Exchange (v1) — feature index

> [!info] **Status:** Draft / awaiting mob review
> Share an assistant-built HTML mini-app via link + QR; recipients preview, see capabilities, and install — re-granting through the existing approval flow.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]]
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

Issues are grouped into **waves** by dependency depth. All issues in a wave can be picked up simultaneously by different devs. Mobile lanes work against [[api-contract]] with a mock, so they are not hard-blocked on backend — the only `blocked-by` edges are genuine build dependencies.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-share-create]] | 60m | backend |
| [[01-shareable-foundation]] | 45m | kmp-common |

### 🟡 Wave 1 — unlocked once wave 0 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[02-share-fetch]] | 30m | backend | `01-share-create` |
| [[03-share-revoke]] | 30m | backend | `01-share-create` |
| [[02-share-action]] | 60m | kmp-common | `01-shareable-foundation` |
| [[03-install-preview]] | 75m | kmp-common | `01-shareable-foundation` |

### 🟠 Wave 2 — final slices

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[04-stop-sharing]] | 30m | kmp-common | `02-share-action` |
| [[01-android-deep-link]] | 45m | android | `03-install-preview` |
| [[01-ios-deep-link]] | 45m | ios | `03-install-preview` |

---

## All issues

```
issues/
├── backend/
│   ├── 01-share-create.md      (W0)
│   ├── 02-share-fetch.md       (W1)
│   └── 03-share-revoke.md      (W1)
├── kmp-common/
│   ├── 01-shareable-foundation.md  (W0)
│   ├── 02-share-action.md          (W1)
│   ├── 03-install-preview.md       (W1)
│   └── 04-stop-sharing.md          (W2)
├── android/
│   └── 01-android-deep-link.md (W2)
└── ios/
    └── 01-ios-deep-link.md     (W2)
```

> [!note] **How to update this index as work lands**
> When an issue's status changes, update its frontmatter (`status: ready` → `in-progress` → `done`). Use Obsidian search `path:issues tag:#status/ready` to see what's grabbable.

---

## Definition of done (whole feature)

- [ ] All issues have status `done` in their frontmatter.
- [ ] [[open-questions]] has zero unresolved items.
- [ ] [[PRD]] has at least one success metric. ✅ (three defined)
- [ ] [[api-contract]] has zero `TBD` markers. ✅ (none; the one numeric guess — size cap — is tracked in [[open-questions#Q1]] and has a working default)
- [ ] End-to-end flow runs on a real device: share on one device → scan QR on another → preview → install → approve → use.
