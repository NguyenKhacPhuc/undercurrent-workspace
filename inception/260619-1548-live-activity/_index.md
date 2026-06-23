---
type: feature-index
feature: live-activity
status: draft
created: 2026-06-19
tags:
  - inception/index
  - feature/live-activity
  - status/draft
---

# Live Activity (v1) — feature index

> [!info] **Status:** Draft / awaiting mob review
> Replace the static "Thinking…" with an animated, narrated in-turn indicator and a friendly per-reply step trail. Client-only; reshapes signal that already reaches the chat.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] (no backend)
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

Single lane (`kmp-common`), so "parallel" here means two devs can grab independent slices without colliding. Wave 0 holds the two independent foundations; Wave 1 builds the meaningful layers on top.

> [!tip]
> Wave = 0 if `blocked-by` is empty, else `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-tool-phrase-map]] | 45m | kmp-common |
| [[02-animated-activity-indicator]] | 60m | kmp-common |

### 🟡 Wave 1 — unlocked once wave 0 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[03-live-tool-narration]] | 60m | kmp-common | `01`, `02` |
| [[04-friendly-step-trail]] | 60m | kmp-common | `01` |

---

## All issues

```
issues/
└── kmp-common/
    ├── 01-tool-phrase-map.md            (W0 — friendly description per action + fallback)
    ├── 02-animated-activity-indicator.md (W0 — animated indicator + dead-air rotation)
    ├── 03-live-tool-narration.md        (W1 — narrate the current action)
    └── 04-friendly-step-trail.md        (W1 — persistent per-reply step record)
```

> [!note] **Build order delivers value incrementally:** `02` alone already kills the "frozen and boring" problem (animated). `01`+`03` make it *meaningful* (says what it's doing). `04` makes it *transparent* (says what it did).

---

## Definition of done (whole feature)

- [ ] All issues have status `done` in their frontmatter. *(1/4 — [[01-tool-phrase-map]] merged PR #49, 2026-06-23.)*
- [ ] [[open-questions]] has zero unresolved items (Q1–Q4: timing, copy, density, actionless-reply behavior).
- [ ] [[PRD]] has at least one success metric. ✅ (two defined)
- [ ] [[api-contract]] has zero `TBD` markers. ✅ (no backend)
- [ ] On a real device: a tool-heavy turn shows the animated indicator narrating each step, then leaves a compact trail above the answer; an off-the-record turn shows nothing.
