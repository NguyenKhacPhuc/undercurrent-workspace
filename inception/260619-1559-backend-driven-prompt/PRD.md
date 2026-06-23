---
type: prd
feature: backend-driven-prompt
status: draft
created: 2026-06-19
tags:
  - inception/prd
  - feature/backend-driven-prompt
  - status/draft
---

# PRD: Backend-driven prompt (v1)

> [!info] **Status:** Draft / awaiting mob review · **Driver:** Phuc · **Last updated:** 2026-06-19
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

Move the assistant's base prompt out of the app binary and onto the backend, so the team can change how the assistant behaves without shipping an app release.

## Problem

Today the assistant's base prompt — its identity, behavioral rules, and anti-hallucination discipline — is a constant compiled into the app. Tuning a single sentence ("stop doing X", "always do Y") means a full build, store review, and waiting for users to update. That's days-to-weeks of latency on the one lever that most directly shapes assistant quality, and it means a bad behavioral regression can't be hotfixed.

Moving the base prompt to the backend turns that lever into a near-instant, central control: edit once on the server, and every client picks it up on next launch. The engine already accepts an injected prompt, so the missing pieces are a place to store/serve it and a client path to fetch and apply it.

**Who has the problem:** the team tuning assistant behavior (today blocked on release cycles), and indirectly every user who waits for a behavioral fix to ship.

## Goals

Testable goals.

- [ ] The assistant's base prompt is served by the backend, not read from a compiled-in constant.
- [ ] An authorized operator can change the served prompt, and clients pick up the change without an app update.
- [ ] On launch, the app fetches the current prompt and uses it for that session's replies.
- [ ] When offline, the app uses the last prompt it successfully fetched.
- [ ] On a fresh install with no fetched prompt yet, the app waits (with clear feedback) until it can fetch one before the assistant becomes usable — there is no compiled-in fallback prompt.
- [ ] The same served prompt drives both Android and iOS.

## Non-goals

Promoted to [[out-of-scope]].

- Making personas, the writer-agent bias, creator-mode instructions, or tool descriptions backend-driven (v1 is the base preamble only).
- Per-user or per-cohort prompts, A/B tests, staged rollouts (v1 is one global config for everyone).
- Mid-session live prompt refresh (changes apply on next launch, not mid-conversation).
- An admin/editor UI for the prompt (v1 update path is a protected operator action, not a product surface).
- Versioned prompt history / rollback UI (beyond the single current config + its revision marker).

## User stories

### Story 1 — The assistant runs on the served prompt

**As a** user, **I want** the assistant to behave according to the latest prompt the team has set, **so that** behavioral fixes reach me without waiting for an app update.

**Acceptance criteria:**
- [ ] On launch, the app fetches the current base prompt from the backend.
- [ ] The assistant's replies in that session are produced using the fetched prompt.
- [ ] If a newer prompt was set since the last launch, the new behavior is in effect after the app is reopened.
- [ ] The behavior is identical on Android and iOS for the same served prompt.

### Story 2 — Offline and cold-start behavior

**As a** user opening the app without a good connection, **I want** it to behave predictably, **so that** I'm not left confused or stuck.

**Acceptance criteria:**
- [ ] If the app has previously fetched a prompt and is now offline, it uses the last fetched prompt and the assistant works normally.
- [ ] On a fresh install with no previously fetched prompt and no connection, the assistant is not usable yet; the app shows a clear "getting set up / couldn't connect — retry" state instead of silently using a built-in prompt.
- [ ] From that blocked state, regaining connection (or retrying) lets the fetch complete and the assistant becomes usable, without needing a manual restart.

### Story 3 — Changing the prompt centrally

**As a** team member tuning the assistant, **I want** to change the served prompt, **so that** I can adjust behavior centrally and see it reach clients on their next launch.

**Acceptance criteria:**
- [ ] An authorized operator can replace the current base prompt with new text.
- [ ] After a change, a client that launches afterward fetches and uses the new prompt.
- [ ] An unauthorized caller cannot change the prompt.
- [ ] The served prompt is seeded with today's current built-in prompt text, so behavior is unchanged at the moment of cut-over.

## Success metrics

How we know this worked, after launch. At least one.

- **Release-free prompt change:** at least one base-prompt change is made and verified to reach clients on next launch **without** an app store release, after launch.
- **Time-to-change:** the elapsed time from "operator edits the prompt" to "a relaunched client uses it" is minutes, not the days-to-weeks an app release takes.
- **No cold-start regression in the field:** the rate of launches that get stuck unable to fetch the prompt stays negligible (watch this closely given the no-fallback decision).

## Constraints

- **Backend + both mobile platforms.** Net-new backend config endpoint + a client fetch/cache path; no substrate change (the engine already accepts an injected base prompt).
- **No compiled-in fallback** (driver decision [[decisions#D4]]): the backend is the single source of truth; first launch must fetch before the assistant is usable. This raises a hard availability dependency — see [[open-questions#Q1]], [[open-questions#Q2]].
- **Single source of truth = no drift**, but also = a bad served prompt affects all clients; a safety check on what gets applied is in scope to discuss ([[open-questions#Q2]]).
- **Fetch happens after sign-in** — the app already gates first launch behind sign-in; the prompt fetch is authorized with the existing session token and runs in that same startup gate.

## Links

- API contract: `./api-contract.md`
- Decisions: `./decisions.md`
- Open questions: `./open-questions.md`
- Out of scope: `./out-of-scope.md`
- Project-wide context: `../../CONTEXT.md`
- Issues: `./issues/`
