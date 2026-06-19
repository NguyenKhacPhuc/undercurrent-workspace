---
type: feature-index
feature: ios-mini-app-render
status: ready
created: 2026-06-19
tags:
  - inception/index
  - feature/ios-mini-app-render
  - status/ready
---

# HTML mini-apps render on iOS — feature index

> [!info] **Status:** Draft / awaiting mob review
> Lightweight, single-story feature. Open questions resolved; ready to construct.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] *(no BE work)*
- Decisions: [[decisions]] *(D1–D3)*
- Open questions: [[open-questions]] *(0 open — all resolved 2026-06-19)*
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

> [!note]
> This is a deliberately **single-story** feature — a host-side lift +
> iOS wiring that delivers one user-observable outcome ("saved HTML
> mini-apps open and run on iOS"). The orchestrator lifts atomically, so
> there's nothing to parallelize; splitting it would create
> non-user-observable sub-stories. One PR in `undercurrent/`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-ios-html-mini-apps-render]] | 90m | kmp-common |

---

## All issues

```
issues/
└── kmp-common/
    └── 01-ios-html-mini-apps-render.md   (W0)
```

> [!note] **How to update this index as work lands**
> Flip the issue's `status` (`ready` → `in-progress` → `done`) in its
> frontmatter. The PR lands in `undercurrent/` (no `weft` change).

---

## Definition of done (whole feature)

- [x] [[01-ios-html-mini-apps-render]] has status `done`. *(Merged — PR #46, 2026-06-19.)*
- [x] [[open-questions]] has zero unresolved items. *(All resolved 2026-06-19.)*
- [x] [[PRD]] has at least one success metric. *(two — open success, parity.)*
- [x] [[api-contract]] has zero `TBD` markers. *(No BE work.)*
- [ ] [[decisions]] reviewed by the mob.
- [x] Both targets compile (`compileDebugKotlinAndroid` +
      `compileKotlinIosSimulatorArm64`) and `:feature:miniapps` tests pass,
      with no Android regression. *(Verified in PR #46.)*
- [ ] A saved HTML mini-app opens → (first-run consent) → renders → runs a
      scope-gated action → persists state, in the **iOS simulator**.
      *(Reviewer smoke; real-device deferred — [[decisions]] D3.)*
