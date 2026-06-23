---
type: prd
feature: live-activity
status: draft
created: 2026-06-19
tags:
  - inception/prd
  - feature/live-activity
  - status/draft
---

# PRD: Live Activity (v1)

> [!info] **Status:** Draft / awaiting mob review · **Driver:** Phuc · **Last updated:** 2026-06-19
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

Replace the static "Thinking…" label with an animated, narrated indicator that shows what the assistant is doing while you wait — and leaves a friendly record of the steps it took.

## Problem

While the assistant works on a reply, the chat shows a single motionless word: "Thinking…". For a quick text answer that's tolerable; for the turns Undercurrent is built around — opening the map, remembering a fact, authoring a whole mini-app — the wait is long and the screen looks frozen. The user can't tell whether it's working, stuck, or hung, and there's nothing to hold attention. It reads as broken and it's boring.

The frustrating part: the app already *knows* what the assistant is doing moment-to-moment — it receives real-time signals as each tool starts and finishes — but it either hides that or shows it as raw, technical, motionless text. The waiting time is dead space that could be a living, reassuring window into the work.

**Who has the problem:** every user, on every non-trivial turn — most acutely on the long, tool-heavy turns that are the product's signature.

## Goals

Testable goals.

- [ ] While a reply is being prepared, the waiting indicator is animated (in motion), not a static word.
- [ ] When the assistant takes an action mid-turn, the indicator shows what it's doing in plain, friendly language (never a raw technical tool name).
- [ ] After the turn, the user can see a friendly record of the steps the assistant took, attached to that reply.
- [ ] When an action fails, it's described in human terms, not as a raw error line.
- [ ] During a genuinely quiet stretch (waiting for the model, gaps between steps), the indicator stays alive with gentle rotating hints rather than freezing.
- [ ] None of this appears for off-the-record turns — only for visible chat replies.

## Non-goals

Promoted to [[out-of-scope]].

- Persona-flavored copy or coloring of the waiting state (deferred — cheap later add).
- Showing the model's internal reasoning / "thinking" tokens (requires substrate work).
- A redesign of the existing detailed trace/audit screen — this is the in-chat, glanceable layer, not that.
- Per-turn analytics surfaced to the user (timings, token counts).
- Configurable / user-tunable animation or verbosity settings.

## User stories

### Story 1 — The wait feels alive

**As a** person waiting on a reply, **I want** the waiting state to be in motion and to keep changing, **so that** I can tell the app is working and don't feel like it froze.

**Acceptance criteria:**
- [ ] As soon as a reply starts being prepared, an animated indicator appears in place of the old static "Thinking…".
- [ ] During a quiet stretch with nothing specific happening yet, the indicator gently rotates through short hint lines instead of holding one frozen word.
- [ ] The rotating hints only begin after a brief quiet threshold, change at a calm pace, and never flicker or jump rapidly.
- [ ] Once the reply's text starts arriving, the indicator gives way to the streaming reply (the existing streaming behavior is preserved).

### Story 2 — See what the assistant is doing

**As a** person waiting on a tool-heavy reply, **I want** to see what the assistant is doing right now in plain language, **so that** the wait is reassuring and I trust what's happening.

**Acceptance criteria:**
- [ ] When the assistant takes an action, the indicator shows a friendly present-tense description of it (e.g. "Looking at the map…", "Building your mini-app…").
- [ ] A raw technical action name is never shown to the user; an unrecognized action falls back to a sensible generic phrase (e.g. "Working…").
- [ ] As the assistant moves from one action to the next, the indicator transitions smoothly between descriptions rather than snapping.
- [ ] If an action fails, the indicator describes it in human terms (e.g. "Couldn't reach the map") rather than showing an error code or stack text.

### Story 3 — See what the assistant did

**As a** person reading a reply, **I want** a brief, friendly record of the steps the assistant took to produce it, **so that** I understand and trust how it got there.

**Acceptance criteria:**
- [ ] After the turn, the steps the assistant took remain attached to that reply as a compact, low-key trail above the answer (e.g. "✓ Looked at the map · ✓ Built your mini-app").
- [ ] The trail uses the same friendly language as the live indicator, in past tense.
- [ ] A failed step is shown humanely in the trail, distinguishable from a successful one, without raw error text.
- [ ] A reply that involved no actions shows no trail — just the answer.
- [ ] The trail is visually quiet enough that a reply with many steps doesn't become a wall of noise.

## Success metrics

How we know this worked, after launch. At least one.

- **Less abandonment of long turns:** the share of long, tool-bearing turns that the user cancels or navigates away from before completion goes down (proxy for "the wait stopped feeling broken"). Target: a measurable decrease vs. the pre-release baseline.
- **Perceived liveliness (qualitative):** in dogfood/user feedback, the chat waiting experience is described as "alive / clear about what's happening" rather than "stuck / boring".

## Constraints

- **Client-only.** No backend and no substrate change — the real-time action signal (start / done / fail) already reaches the chat layer; this feature reshapes and animates it.
- **Visible turns only.** Off-the-record turns (the ephemeral mini-app → assistant path) must remain hidden and must not surface any activity in the visible chat.
- **Reasoning tokens are unavailable** to the chat layer today; v1 narrates *actions*, not the model's private thinking.

## Links

- API contract: `./api-contract.md` (no backend)
- Decisions: `./decisions.md`
- Open questions: `./open-questions.md`
- Out of scope: `./out-of-scope.md`
- Project-wide context: `../../CONTEXT.md`
- Issues: `./issues/`
