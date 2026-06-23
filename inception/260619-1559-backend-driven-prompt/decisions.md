---
type: decisions
feature: backend-driven-prompt
created: 2026-06-19
tags:
  - inception/decisions
  - feature/backend-driven-prompt
---

# Decisions — Backend-driven prompt

> [!info] Unilateral driver decisions to ratify in mob review, with rationale.

## D1 — Scope: the base preamble only

**Decision:** Only the base prompt (app intro + behavioral / anti-hallucination rules — today's `ASSISTANT_APP_PREAMBLE`) becomes backend-driven. Personas, writer-agent bias, creator-mode instructions, and tool descriptions stay compiled-in for v1.

**Why:** The base preamble is the highest-leverage behavior lever and the cleanest single thing to externalize (it's already injected through one parameter). The others can follow the same pattern later without rework. Keeps v1 a tight vertical slice.

## D2 — One global config for everyone

**Decision:** The backend serves a single current prompt to all clients. No per-user, per-cohort, A/B, or staged rollout.

**Why:** Matches the stated need — "change it dynamically and centrally." Targeting/experimentation is a large separate build (assignment, metrics) and isn't required to remove the release-cycle bottleneck.

## D3 — Apply at startup, in effect next launch

**Decision:** The client fetches at launch and uses the fetched prompt for that session. A change made on the backend reaches a client on its next app open. No mid-session refresh.

**Why:** The base prompt is cached at the model's static tier; swapping it mid-conversation would thrash that cache and could make one conversation behave inconsistently across turns. Next-launch application is predictable and cache-friendly.

## D4 — Cached-only, no compiled-in fallback

**Decision:** The backend is the single source of truth. The client uses the last successfully fetched prompt, and prefers it when offline. There is **no** compiled-in fallback prompt. On a fresh install with no fetched prompt and no connection, the assistant is unavailable until a fetch succeeds (the app shows a clear blocked/retry state).

**Why:** Eliminates drift between a baked copy and the server copy, and forces the server to be authoritative. 

> [!warning] **Risk the mob must accept.** This creates a hard availability dependency: if the backend is down during a fresh install, or if a broken/empty prompt is served, clients are affected with no local safety net. Mitigations to decide: the cold-start gate UX ([[open-questions#Q1]]) and a server-side guard against applying an empty/invalid prompt ([[open-questions#Q2]]). The seeded initial value (today's prompt) means existing installs are never worse off at cut-over.

## D5 — Fetch is authorized and happens after sign-in

**Decision:** The prompt fetch requires the existing session token and runs as part of the post-sign-in startup, not as a public pre-auth call.

**Why:** The app already gates first launch behind sign-in; reusing the session token avoids exposing the prompt publicly and avoids inventing a second auth path. Ordering: sign-in gate → prompt-config gate → assistant usable.

## D6 — Operator-driven update; no editor UI in v1

**Decision:** Changing the prompt is a protected operator action against a backend endpoint — not a user-facing product feature. No admin editing UI in v1.

**Why:** The goal is release-free change, which a protected endpoint satisfies. A polished editor UI (history, diff, rollback, roles) is a separate, larger effort. The exact operator-auth mechanism is parked ([[open-questions#Q3]]).
