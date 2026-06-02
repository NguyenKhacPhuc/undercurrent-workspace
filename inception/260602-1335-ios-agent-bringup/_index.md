---
type: feature-index
feature: ios-agent-bringup
status: draft
created: 2026-06-02
tags:
  - inception/index
  - feature/ios-agent-bringup
  - status/draft
---

# iOS agent bring-up — feature index

> [!info] **Status:** Draft / awaiting mob review
> Bring undercurrent's iOS app from a "coming-soon" shell to a working agent, by consuming the merged **weft-ios-parity** substrate. MVP = working text agent + OAuth integrations; Koog-bits + voice deferred ([[out-of-scope]]).

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] (no BE work)
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]
- Substrate it consumes: `../260602-0149-weft-ios-parity/`

---

## Parallel work plan

8 stories across two lanes: `kmp-common` (lift the Weft-backed layer to shared code — Android keeps working, iOS gains it) and `ios` (DI wiring + app orchestration in `undercurrent/composeApp/iosMain`).

> [!tip]
> Wave = 0 if `blocked-by` is empty, else `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-share-history-usage-repos]] | 60m | kmp-common |
| [[02-share-secure-key-repo]] | 45m | kmp-common |
| [[03-share-agent-builder]] | 75m | kmp-common |
| [[04-ios-sdk-at-launch]] | 45m | ios |

### 🟡 Wave 1 — unlocked once their blockers land

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[05-share-chat-streaming]] | 90m | kmp-common | `03` |
| [[08-ios-integrations-signin]] | 60m | ios | `02`, `04` |

### 🟠 Wave 2 — the agent goes live

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[06-ios-adopt-shared-layer]] | 60m | ios | `01`, `02`, `04`, `05` |

### 🔵 Wave 3 — app lifecycle parity

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[07-ios-agent-lifecycle]] | 75m | ios | `06` |

> [!note] **Why Waves 2 and 3 are single-issue.**
> Intentional: [[06-ios-adopt-shared-layer]] is the *integration* point ("the agent answers on iOS") — it can only land once the shared lifts (`01`,`02`,`05`) and the runtime (`04`) exist. [[07-ios-agent-lifecycle]] orchestrates on top of a live agent, so it follows `06`. The breadth is front-loaded into Waves 0–1 (6 of 8 issues).

---

## All issues

```
issues/
├── kmp-common/   (undercurrent/ — shared lifts)
│   ├── 01-share-history-usage-repos.md   (W0)
│   ├── 02-share-secure-key-repo.md       (W0)
│   ├── 03-share-agent-builder.md         (W0)
│   └── 05-share-chat-streaming.md        (W1 ← 03)
└── ios/          (undercurrent/ — DI + app)
    ├── 04-ios-sdk-at-launch.md           (W0)
    ├── 08-ios-integrations-signin.md     (W1 ← 02,04)
    ├── 06-ios-adopt-shared-layer.md      (W2 ← 01,02,04,05)
    └── 07-ios-agent-lifecycle.md         (W3 ← 06)
```

> [!note] **Updating this index as work lands**
> Flip each issue's `status` (`ready` → `in-progress` → `done`) in its frontmatter. All PRs land in `undercurrent/` (not weft). Branch from `undercurrent` `origin/main` ([[decisions]] D4).

---

## Definition of done (whole feature)

- [ ] All issues have status `done`.
- [ ] [[open-questions]] has zero unresolved items (Q1 + Q2 confirmed by the mob).
- [ ] [[PRD]] has at least one success metric. ✅ (three)
- [ ] [[api-contract]] has zero `TBD` markers. ✅ (no BE work)
- [ ] An iOS user sends a message and gets a streaming agent reply on a real device; integrations connect via OAuth; the shared layer compiles green on Android + iOS with no Android regression.
