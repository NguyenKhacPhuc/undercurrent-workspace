---
type: feature-index
feature: backend-driven-prompt
status: draft
created: 2026-06-19
tags:
  - inception/index
  - feature/backend-driven-prompt
  - status/draft
---

# Backend-driven prompt (v1) — feature index

> [!info] **Status:** Draft / awaiting mob review
> Move the assistant's base prompt to the backend so behavior can change without an app release. One global config; fetched at startup; cached-only with no compiled-in fallback.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]]
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

Client lanes work against [[api-contract]] with a mock, so they are not hard-blocked on the backend. The only `blocked-by` edges are genuine build dependencies.

> [!tip]
> Wave = 0 if `blocked-by` is empty, else `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-serve-prompt-config]] | 45m | backend |
| [[01-prompt-config-repository]] | 45m | kmp-common |

### 🟡 Wave 1 — unlocked once wave 0 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[02-update-prompt-config]] | 40m | backend | `01-serve-prompt-config` |
| [[02-cold-start-gate]] | 50m | kmp-common | `01-prompt-config-repository` |
| [[01-android-apply-served-prompt]] | 40m | android | `01-prompt-config-repository` |
| [[01-ios-apply-served-prompt]] | 40m | ios | `01-prompt-config-repository` |

---

## All issues

```
issues/
├── backend/
│   ├── 01-serve-prompt-config.md        (W0 — store + serve, seeded with today's prompt)
│   └── 02-update-prompt-config.md       (W1 — operator change endpoint)
├── kmp-common/
│   ├── 01-prompt-config-repository.md   (W0 — fetch + cache + provide; "not ready" state)
│   └── 02-cold-start-gate.md            (W1 — block until a prompt exists, no fallback)
├── android/
│   └── 01-android-apply-served-prompt.md (W1 — build assistant from served prompt)
└── ios/
    └── 01-ios-apply-served-prompt.md    (W1 — build assistant from served prompt)
```

> [!warning] **Cross-lane coordination point.** The per-platform "apply" stories and the cold-start gate together enforce the no-fallback rule: the assistant must not be constructed until a provided prompt exists. Whoever picks up the platform stories should sync with the gate story on construction ordering.

---

## Definition of done (whole feature)

- [ ] All issues have status `done` in their frontmatter. *(5/6 — wave 0: 01-serve-prompt-config (BE #12), 01-prompt-config-repository (#56); wave 1: 02-cold-start-gate (#57), 01-android-apply-served-prompt (#58), 01-ios-apply-served-prompt (#59). Remaining: 02-update-prompt-config (BE #13, open).)*
- [ ] [[open-questions]] has zero unresolved items (Q1 cold-start UX, Q2 invalid-prompt guard, Q3 operator auth, Q4 revision short-circuit).
- [ ] [[PRD]] has at least one success metric. ✅ (three defined)
- [ ] [[api-contract]] has zero `TBD` markers. ✅ (operator-auth mechanism parked as a deferred decision, not a TBD field)
- [ ] End-to-end: an operator changes the prompt → a relaunched client (Android and iOS) uses it → no app release involved.
