---
type: open-questions
feature: backend-driven-prompt
created: 2026-06-19
tags:
  - inception/open-questions
  - feature/backend-driven-prompt
---

# Open questions — Backend-driven prompt

> [!question] For the mob to resolve. Driver guesses give a starting point. Q1 and Q2 are the ones the no-fallback decision ([[decisions#D4]]) makes important.

## Q1 — Cold-start gate UX (the no-fallback consequence)

On a fresh install with no fetched prompt and no/slow network, the assistant is blocked until a fetch succeeds. What exactly does the user see and do?
- Messaging ("Getting set up…" vs "Couldn't connect — Retry").
- Is there a fetch timeout before showing the retry affordance? How long?
- Does it auto-retry on reconnect, or only on a manual tap?

**[DRIVER GUESS: a brief "Getting set up…" spinner; after ~10s without success, switch to "Couldn't connect — Retry"; auto-retry when connectivity returns AND on manual tap.]** Mob: confirm the timing and copy.

## Q2 — Server-side guard on what can be applied

With no client fallback, a bad served prompt (empty, truncated, accidental) affects everyone. Should the backend refuse to store an invalid prompt, and what counts as invalid?

**[DRIVER GUESS: reject empty/whitespace and below a minimum length on `PUT`; otherwise trust the operator. No content validation beyond that in v1.]** Mob: is a minimum-length + non-empty guard enough, or do we want a stronger safeguard (e.g. a required confirmation, a server-held last-known-good the client can fall back to)?

## Q3 — Operator-authorization mechanism for updates

How is the `PUT /v1/prompt-config` caller authorized? Options: a shared operator secret (env-configured), a designated admin account flag, or restrict to a manual DB/ops procedure for now.

**[DRIVER GUESS: a server-configured operator secret required on the update endpoint for v1; revisit if/when more operators or an editor UI arrive.]** Mob: confirm the mechanism (this is the one parked item in [[api-contract]]).

## Q4 — Revision check to avoid re-downloading an unchanged prompt

Should the client send its cached `revision` so the server can answer "unchanged" cheaply, or just always return the full prompt at startup?

**[DRIVER GUESS: nice-to-have, not required for v1 — always returning the full payload at startup is fine at this scale; add the revision short-circuit only if payload size becomes a concern.]** Mob: confirm we can skip the short-circuit for v1.
