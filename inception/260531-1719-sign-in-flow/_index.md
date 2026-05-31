---
type: feature-index
feature: sign-in-flow
status: superseded
created: 2026-05-31
superseded: 2026-06-01
tags:
  - inception/index
  - feature/sign-in-flow
  - status/superseded
---

# Sign-in flow — feature index

> [!warning] **Status: SUPERSEDED — 2026-06-01.**
> This Inception was specced under the explicit assumption that **no backend exists** (see `Constraints` in [[PRD]]: "No backend. All persistence is local."). The backend-bootstrap-auth Inception (`260531-1733-backend-bootstrap-auth/`) shipped end-to-end on 2026-06-01, which inverts the premise: profile + auth are now BE-owned.
>
> All 3 stories below are kept as a record of pre-BE thinking but are **not buildable as written** — most ACs no longer match the system shape. The replacement Inception is `260601-0040-mobile-auth-wiring/`, which specs the BE-backed register / sign-in / `/v1/me` / sign-out client flows.
>
> See [[decisions#D5 (added 2026-06-01)]] for why this was superseded rather than revised in place.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] *(no BE work)*
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[../../CONTEXT]]

---

## Parallel work plan

Issues are grouped into **waves** by dependency depth. All issues in a wave can be picked up simultaneously by different devs.

> [!tip]
> Compute wave for each issue: 0 if `blocked-by` is empty, otherwise `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-user-profile-store]] | 60m | kmp-common |

### 🟡 Wave 1 — unlocked once Wave 0 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[02-first-launch-sign-in-screen]] | 90m | kmp-common | `01` |
| [[03-edit-profile-from-settings]] | 60m | kmp-common | `01` |

---

> [!warning] **Parallelization sanity check**
> Wave 0 contains a single small story by design. The store is a foundation slice (entity + repository interface + DataStore-Preferences impl); both Wave 1 stories depend on it. Once Wave 0 lands — same day, by estimate — Wave 1 fans out and two devs can grab `02` and `03` in parallel without touching the same files. The total feature is intentionally small (~3.5 hours of focused TDD across all three stories), so the serial Wave 0 → Wave 1 step is acceptable.

---

## All issues

```
issues/
└── kmp-common/
    ├── 01-user-profile-store.md
    ├── 02-first-launch-sign-in-screen.md
    └── 03-edit-profile-from-settings.md
```

No `android/` or `ios/` lane directories — all work is commonMain. See [[decisions#D4]].

> [!note] **How to update this index as work lands**
> When an issue's status changes, update its frontmatter (`status: ready` → `in-progress` → `done`). The Properties panel and tag pane will reflect it. Use Obsidian's search `path:issues tag:#status/ready` to see what's grabbable.

---

## Definition of done (whole feature)

- [ ] All issues have status `done` in their frontmatter.
- [ ] [[open-questions]] has zero unresolved items.
- [ ] [[PRD]] has at least one success metric. *(currently: two — completion rate + edit usage)*
- [ ] If applicable, [[api-contract]] has zero `TBD` markers. *(currently: file says "no BE work" — trivially satisfied)*
- [ ] End-to-end flow runs on a real device for both platforms: fresh-install path lands on sign-in, completes, persists across kill; Settings edits the values; values still there after another kill.
