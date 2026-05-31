---
type: open-questions
feature: sign-in-flow
created: 2026-05-31
tags:
  - inception/open-questions
  - feature/sign-in-flow
---

# Open questions

> [!question]
> Driver's parking lot. Anything the driver could not resolve alone goes here for the mob.
>
> **The Inception phase ends when this file is empty** (or only contains items the mob explicitly deferred to a later phase).

## Open

### Q1 — Email validation rules?

- **Why it matters:** Drives the AC of the sign-in story and the profile editor. Tighter validation = fewer junk profiles but more friction; looser = more permissive but garbage in. Affects whether Construction wires up a format validator vs just "non-empty".
- **[DRIVER GUESS]:** Basic format check (must contain `@` and a `.` after it, no spaces). Required. No length cap beyond what the input field naturally allows.
- **[ASKED OF]:** Product

### Q2 — Display name validation?

- **Why it matters:** Same trade-off as Q1. Also: do we allow emoji? Single-character names?
- **[DRIVER GUESS]:** Non-empty after trim. Cap at 40 characters (matches the Conversation title cap in [[CONTEXT#Conversation]]). Allow unicode incl. emoji.
- **[ASKED OF]:** Product

### Q3 — Where does the captured display name actually surface in the app?

- **Why it matters:** Drives whether we need follow-up stories now (e.g. agent system-prompt injection, home-surface greeting) or whether collection alone is the whole feature for v1. The PRD currently lists this as a non-goal — confirming with the mob.
- **[DRIVER GUESS]:** Not surfaced anywhere in v1 beyond Settings itself. The point of v1 is collection for future use. Surfacing in agent prompt / greetings is a separate Inception.
- **[ASKED OF]:** Product

### Q4 — Privacy / "where does this go" disclosure on the sign-in screen?

- **Why it matters:** The user is being asked for an email on a blocking screen with no obvious reason ("why does this app need my email?"). A short reassurance reduces drop-off and is honest about the local-only stance.
- **[DRIVER GUESS]:** Single line of helper text under the email field: "Stored only on this device. We don't send your email anywhere yet."
- **[ASKED OF]:** Product / Legal

### Q5 — When backend sync eventually ships, how does the existing local profile migrate?

- **Why it matters:** Choosing "local-only, no future-proofing" today (see [[decisions#D2]]) means this question is deferred — but the mob may want to push back ("at least reserve a stable id?"). If we don't reserve one, BE-onboarding can't link a fresh account back to the user's existing local data.
- **[DRIVER GUESS]:** Defer entirely. When BE ships, the BE Inception will spec the migration; if it needs a local id, that Inception will add one and the migration will generate it on the spot.
- **[ASKED OF]:** All — re-asked at start of BE Inception.

### Q6 — How does the sign-in screen interleave with the existing first-run onboarding (provider picker / API key)?

- **Why it matters:** Today, a fresh user lands on a flow that asks them to pick an LLM provider and paste an API key. Adding a blocking sign-in step needs an order. Sign-in first, then provider? Provider first, then sign-in? Combined into one onboarding flow?
- **[DRIVER GUESS]:** Sign-in first (it's faster and more universal), then the existing provider/API-key step. Sign-in is its own screen; no merging.
- **[ASKED OF]:** Android + iOS (whoever owns the current onboarding shell knows best where to splice).

### Q7 — Can the user sign out / reset the profile?

- **Why it matters:** PRD covers edit but not delete. With no backend, "sign out" semantically means "wipe local profile and re-show sign-in". May or may not be a v1 ask.
- **[DRIVER GUESS]:** Not in v1. Reinstalling the app is the escape hatch. Add a "Reset profile" entry in Settings as a separate small follow-up if anyone needs it.
- **[ASKED OF]:** Product

## Resolved

Move resolved questions here with the answer and the date. Don't delete — they become a record.
