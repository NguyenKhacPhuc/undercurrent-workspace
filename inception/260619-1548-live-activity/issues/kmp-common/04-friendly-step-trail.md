---
type: issue
feature: live-activity
lane: kmp-common
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 1
estimate: 60m
blocked-by: ["[[01-tool-phrase-map]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/live-activity
  - status/in-progress
  - wave/1
---

# [kmp-common] A reply keeps a friendly record of the steps that produced it

**Lane:** kmp-common
**PRD section:** [[PRD#Story 3 — See what the assistant did]]
**API contract section:** n/a (client-only)

## Why

Transparency after the fact: once a turn finishes, the user can see — compactly and in plain language — what the assistant did to get to the answer. Replaces the raw "→ tool / ✓ tool" lines with a quiet, friendly trail above the reply.

## Acceptance criteria

- [ ] After a turn that involved actions, the steps remain attached to that reply as a compact trail above the answer (e.g. "✓ Looked at the map · ✓ Built your mini-app").
- [ ] The trail uses the same friendly language as the live indicator, in past tense.
- [ ] A failed step is shown humanely and is visually distinguishable from a successful one, with no raw error text.
- [ ] A reply that involved no actions shows no trail — just the answer.
- [ ] The trail is visually quiet (low-contrast, compact) so a reply with many steps does not become a wall of noise.
- [ ] The trail appears only for visible chat replies — never for off-the-record turns.

## Blocked by

- [[01-tool-phrase-map]] — needs the friendly past-tense + failure descriptions.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`).

- **Existing pattern to mirror:** the current rendering of tool start/done/fail as displayed messages — this story reshapes those into the compact per-reply trail rather than inline raw lines.
- **Watch out for:** density is the known risk ([[open-questions#Q3]]) — keep it to a low-key single line that wraps; the detailed trace screen remains the place for depth. Don't double-render (raw lines AND the trail).

## Out of scope for this story

- Tapping a step to drill into detail (a later idea; the trace screen covers depth today).
- The in-flight live indicator ([[02-animated-activity-indicator]], [[03-live-tool-narration]]).
