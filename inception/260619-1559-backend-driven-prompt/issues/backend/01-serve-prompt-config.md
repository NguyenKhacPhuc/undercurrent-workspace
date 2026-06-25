---
type: issue
feature: backend-driven-prompt
lane: backend
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/backend
  - feature/backend-driven-prompt
  - status/in-progress
  - wave/0
---

# [Backend] The backend serves the current base prompt

**Lane:** Backend
**PRD section:** [[PRD#Story 1 — The assistant runs on the served prompt]]
**API contract section:** [[api-contract#`GET /v1/prompt-config` — fetch the current base prompt]]

## Why

Foundation: there must be one place that holds the assistant's current base prompt and hands it to signed-in clients. Seeded with today's built-in prompt text so the cut-over changes nothing about behavior.

## Acceptance criteria

- [ ] A signed-in client can fetch the current base prompt, along with a revision marker and a last-updated time.
- [ ] The stored prompt is seeded with the current built-in prompt text, so the first served value matches today's behavior exactly.
- [ ] A request without a valid session is refused.
- [ ] If the prompt has somehow not been seeded, the client gets a clear "temporarily unavailable" response rather than an empty or broken prompt.

## Blocked by

- nothing — independently grabbable.

## Hints (non-binding)

> [!tip]
> Construction reads `backend/CLAUDE.md`. Mirror the auth endpoints for the response envelope, bearer validation, and error codes.

- **Likely work:** a single-row config store (the prompt text + a revision + updated-at) and the authorized `GET` route. Seed the row with the exact current `ASSISTANT_APP_PREAMBLE` text (lives in the app today — coordinate to copy it verbatim at cut-over).
- **Watch out for:** the revision marker must change whenever the text changes (the update story relies on it).

## Out of scope for this story

- The update endpoint ([[02-update-prompt-config]]).
- Any client fetch/cache behavior (kmp-common).
