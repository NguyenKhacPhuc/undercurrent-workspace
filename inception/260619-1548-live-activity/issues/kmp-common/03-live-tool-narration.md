---
type: issue
feature: live-activity
lane: kmp-common
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 1
estimate: 60m
blocked-by: ["[[01-tool-phrase-map]]", "[[02-animated-activity-indicator]]"]
tags:
  - inception/issue
  - lane/kmp-common
  - feature/live-activity
  - status/in-progress
  - wave/1
---

# [kmp-common] The indicator shows what the assistant is doing right now

**Lane:** kmp-common
**PRD section:** [[PRD#Story 2 — See what the assistant is doing]]
**API contract section:** n/a (client-only)

## Why

This is the meaningful layer: instead of generic hints, the live indicator narrates the assistant's *actual* current action in plain language — driven by the start/done/fail signal that already flows to the chat — so the wait is reassuring and trustworthy.

## Acceptance criteria

- [ ] When the assistant starts an action, the indicator shows its friendly present-tense description (e.g. "Looking at the map…", "Building your mini-app…").
- [ ] As the assistant moves from one action to the next, the indicator transitions smoothly between descriptions rather than snapping.
- [ ] A raw technical action name is never shown; an unrecognized action shows the generic fallback description.
- [ ] If an action fails, the indicator describes it in human terms (e.g. "Couldn't reach the map"), not as an error code or raw message.
- [ ] When no specific action is in progress, the indicator falls back to the rotating dead-air hints from the previous story.
- [ ] This narration appears only for visible chat replies — never for off-the-record turns.

## Blocked by

- [[01-tool-phrase-map]] — needs the friendly descriptions.
- [[02-animated-activity-indicator]] — extends that indicator surface.

## Hints (non-binding)

> [!tip]
> Verify includes both target compiles + module `test` (`undercurrent/CLAUDE.md`). The live action signal already arrives as tool start/done/fail chunks in the chat stream — reshape it, don't re-plumb it.

- **Existing pattern to mirror:** how tool start/done/fail chunks currently surface into the chat's displayed messages (the existing raw "→ tool" rendering) — this story redirects that signal into the indicator's current-activity line.
- **Watch out for:** several actions can occur in one turn; the indicator should reflect the *current* one and hand back to dead-air hints between them.

## Out of scope for this story

- The after-turn persistent record ([[04-friendly-step-trail]]).
