---
type: feature-index
feature: weft-ios-parity
status: ready
created: 2026-06-02
tags:
  - inception/index
  - feature/weft-ios-parity
  - status/ready
---

# Weft iOS parity — feature index

> [!success] **Status:** In construction — 13/15 stories merged to `weft` main (2026-06-02). [[open-questions]] Q3 resolved (deferred) 2026-06-19; the only remaining gaps are the deferred voice story + on-device smoke.
> **Done & merged:** all 10 Wave-0 substrate stories (01–10) + 12 (OAuth) + 14 (one-call setup) + 13 (devtools-iOS). See each issue's `merged-pr`.
> **Remaining:** 11 (voice — **deferred**, AVAudioSession K/N binding gap, see [[open-questions]] Q3) + on-device smoke (deferred — no hardware 2026-06-19). `ios/01` (host adoption) is **superseded** → re-inceptioned as the full iOS agent bring-up at `inception/260602-1335-ios-agent-bringup/` (the host turned out to be a coming-soon shell, not an adoption tweak).
> Make the Weft substrate set up and run on iOS with the same one-call mechanism Android has. Foundation/mechanism scope — ~20 device capabilities deliberately deferred ([[out-of-scope]]).

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] (no BE work)
- Decisions: [[decisions]]
- Open questions: [[open-questions]]
- Out of scope: [[out-of-scope]]
- Project-wide context: [[CONTEXT]]

---

## Parallel work plan

15 issues across two lanes (`substrate` in `weft/`, `ios` in `undercurrent/`). The shape is **"leaves first, integrate last"**: ten independent slices land in Wave 0, three platform flows in Wave 1, then a single integration story and a single host-adoption story.

> [!tip]
> Wave = 0 if `blocked-by` is empty, else `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

Ten independently grabbable slices. The `os-bridge` capability slices touch separate files, so multiple devs can work them in parallel without contention.

| Issue | Estimate | Lane |
|---|---|---|
| [[01-ios-shared-sdk-composition]] | 120m | substrate |
| [[02-ios-credential-vault]] | 45m | substrate |
| [[03-ios-permission-prompts]] | 90m | substrate |
| [[04-ios-clipboard]] | 20m | substrate |
| [[05-ios-open-links]] | 20m | substrate |
| [[06-ios-haptics]] | 30m | substrate |
| [[07-ios-screen-power]] | 30m | substrate |
| [[08-ios-image-editing]] | 60m | substrate |
| [[09-ios-system-info]] | 45m | substrate |
| [[10-ios-share-sheet]] | 45m | substrate |

### 🟡 Wave 1 — unlocked once their blockers land

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[11-ios-voice-input]] | 90m | substrate | `03` |
| [[12-ios-provider-signin]] | 75m | substrate | `02` |
| [[13-ios-debug-overlay]] | 120m | substrate | `01` |

### 🟠 Wave 2 — integration point

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[14-ios-one-call-setup]] | 60m | substrate | `01`–`11` (composition + all foundational capabilities) |

### 🔵 Wave 3 — host adoption (cross-repo)

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[01-host-adopt-substrate-capabilities]] | 90m | ios | `14`, `02`, `11` |

> [!note] **Why Waves 2 and 3 are single-issue.**
> This is intentional, not a planning smell. [[14-ios-one-call-setup]] is the *integration* point — it assembles the composition + every foundational capability into the one-call setup, so by definition it waits on them. [[01-host-adopt-substrate-capabilities]] is the *consumption* point — undercurrent can't adopt the one-call setup until it exists, and it lives in a separate repo (`undercurrent/`) that must bump the `weft/` submodule first ([[decisions]] D3). The parallelism is front-loaded into Waves 0–1 (13 of 15 issues) where the actual breadth of work is.

---

## All issues

```
issues/
├── substrate/   (weft/ — 14 issues)
│   ├── 01-ios-shared-sdk-composition.md      (W0)
│   ├── 02-ios-credential-vault.md            (W0)
│   ├── 03-ios-permission-prompts.md          (W0)
│   ├── 04-ios-clipboard.md                   (W0)
│   ├── 05-ios-open-links.md                  (W0)
│   ├── 06-ios-haptics.md                     (W0)
│   ├── 07-ios-screen-power.md                (W0)
│   ├── 08-ios-image-editing.md               (W0)
│   ├── 09-ios-system-info.md                 (W0)
│   ├── 10-ios-share-sheet.md                 (W0)
│   ├── 11-ios-voice-input.md                 (W1 ← 03)
│   ├── 12-ios-provider-signin.md             (W1 ← 02)
│   ├── 13-ios-debug-overlay.md               (W1 ← 01)
│   └── 14-ios-one-call-setup.md              (W2 ← 01–11)
└── ios/         (undercurrent/ — 1 issue)
    └── 01-host-adopt-substrate-capabilities.md (W3 ← 14,02,11)
```

> [!note] **Updating this index as work lands**
> Flip each issue's `status` (`ready` → `in-progress` → `done`) in its frontmatter. Search `path:issues tag:#status/ready` for what's grabbable. Substrate issues are PRs in `weft/`; the ios issue is a PR in `undercurrent/`.

---

## Definition of done (whole feature)

- [ ] All issues have status `done`. *(11-ios-voice-input deferred — K/N AVAudioSession binding gap, see open-questions Q3; 01 superseded by ios-agent-bringup; all other 13 done)*
- [x] [[open-questions]] has zero unresolved items — Q1 + Q2 resolved 2026-06-02 (see [[decisions]] D5 + [[13-ios-debug-overlay]]).
- [x] [[decisions]] reviewed and ratified by the mob — 2026-06-02 (D1–D5).
- [ ] [[PRD]] has at least one success metric. ✅ (three)
- [ ] [[api-contract]] has zero `TBD` markers. ✅ (no BE work)
- [ ] An iOS host stands up the SDK from one call and runs an agent turn on a real device, with all 10 foundational capabilities working and undercurrent's duplicated impls deleted. *(deferred — requires on-device smoke; no hardware available 2026-06-19)*
