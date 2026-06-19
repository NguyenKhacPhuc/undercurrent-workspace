---
type: feature-index
feature: ios-mini-app-send-message
status: ready
created: 2026-06-19
tags:
  - inception/index
  - feature/ios-mini-app-send-message
  - status/ready
---

# Mini-apps can ask the assistant on iOS — feature index

> [!info] **Status:** Draft / awaiting mob review
> Lightweight single-story follow-up to `260619-1125-ios-mini-app-render`.
> Closes the last iOS mini-app parity gap. Open questions resolved.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] *(no BE work)*
- Decisions: [[decisions]] *(D1–D3)*
- Open questions: [[open-questions]] *(0 open)*
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]
- Predecessor: `../260619-1125-ios-mini-app-render/`

---

## Parallel work plan

> [!note]
> Single-story feature — one shared handler + an iOS DI binding,
> delivering one user-observable outcome ("a mini-app can ask the
> assistant on iOS"). Nothing to parallelize. One PR in `undercurrent/`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-ios-mini-app-ask-assistant]] | 45m | kmp-common |

---

## All issues

```
issues/
└── kmp-common/
    └── 01-ios-mini-app-ask-assistant.md   (W0)
```

> [!note] **How to update this index as work lands**
> Flip the issue's `status` (`ready` → `in-progress` → `done`). The PR
> lands in `undercurrent/` (no `weft` change).

---

## Definition of done (whole feature)

- [x] [[01-ios-mini-app-ask-assistant]] has status `done`. *(Merged — PR #47, 2026-06-19.)*
- [x] [[open-questions]] has zero unresolved items. *(2026-06-19.)*
- [x] [[PRD]] has at least one success metric. *(two — parity, no regression.)*
- [x] [[api-contract]] has zero `TBD` markers. *(No BE work.)*
- [ ] [[decisions]] reviewed by the mob.
- [x] Both targets compile (`compileDebugKotlinAndroid` +
      `compileKotlinIosSimulatorArm64`) and the relevant module tests pass,
      with no Android regression. *(Verified in PR #47.)*
- [ ] A mini-app on iOS asks the assistant and gets a reply, in the
      **iOS simulator**. *(Real-device deferred — predecessor [[decisions]] D3.)*
