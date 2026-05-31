---
type: open-questions
feature: mobile-auth-wiring
created: 2026-06-01
tags:
  - inception/open-questions
  - feature/mobile-auth-wiring
---

# Open questions

> [!question]
> Driver's parking lot. Anything the driver could not resolve alone goes here for the mob.
>
> **The Inception phase ends when this file is empty** (or only contains items the mob explicitly deferred to a later phase).

## Open

### Q1 — Client mirrors BE validation rules exactly?

- **Why it matters:** The BE enforces email (loose format), password (≥8), displayName (non-empty trimmed, ≤40). Mirroring on the client lets us fail fast with inline errors before any submit. Mismatching means the client lets bad input through and the BE returns 400.
- **[DRIVER GUESS]:** Yes, mirror exactly. Encode the rules in one shared `AuthValidation` helper in `kmp-common` so they live in exactly one place; any future change is a one-line edit.
- **[ASKED OF]:** Product / BE (re-confirms BE's resolved Q2)

### Q2 — UX form for register-vs-sign-in mode switch on the same screen?

- **Why it matters:** Drives Story 1 visual + interaction shape. Two distinct screens with a "create account" link? Tab toggle at the top? Segmented control?
- **[DRIVER GUESS]:** A simple toggle at the top of the form: "Sign In | Register". Default = Sign In (most users on a re-install / second-device install). Tapping the toggle swaps modes; common fields (email, password) preserve their values; register adds the displayName field below.
- **[ASKED OF]:** Product / Design

### Q3 — What does "wipe local token + route to sign-in screen" look like *during* an authed user flow?

- **Why it matters:** If a user is mid-conversation and a request returns 401 (because their session got revoked from another device, or expired), what happens to the chat draft / in-flight tool call / etc?
- **[DRIVER GUESS]:** The current screen is interrupted, a brief toast says "You were signed out — please sign in again", and the sign-in screen appears with the email field pre-filled from the last known account. In-flight transient state (typed messages, partial tool calls) is lost. Acceptable for v1; we are NOT building an in-memory queue.
- **[ASKED OF]:** Product

### Q4 — Mobile telemetry for auth events?

- **Why it matters:** PRD declares success metrics ("sign-in completion rate", "authed-401-recoveries-per-week", "sign-out usage") — those need events somewhere.
- **[DRIVER GUESS]:** Reuse the existing trace harness if it can emit non-LLM-turn events; otherwise add a tiny `AuthTelemetry` interface in `kmp-common` with a noop default + a future swap-in for whatever analytics target ships next. Counting only (no email content, no token content).
- **[ASKED OF]:** Android + iOS (whoever owns telemetry today)

### Q5 — First-launch race: what if the user has a stored token but it was revoked / expired server-side?

- **Why it matters:** Story 2 AC says "On any 401 the local token is wiped." But the first launch screens BEFORE the home surface is rendered — if the token is dead, do we route to sign-in immediately, or does the user see a flash of home surface first?
- **[DRIVER GUESS]:** Do a `GET /v1/me` validation call on every cold launch with a stored token. While in flight: show a brief splash / spinner over the would-be home surface (no flash). On 200: render home. On 401: wipe + route to sign-in. On network error: optimistically render home (the next authed call will eventually fail and route correctly).
- **[ASKED OF]:** Product / Android + iOS

### Q6 — Should the BE client retry transient network errors automatically?

- **Why it matters:** A flaky mobile connection produces spurious timeouts; auto-retry hides them. But retry on sign-up could create duplicate accounts; retry on sign-in is generally safe.
- **[DRIVER GUESS]:** No auto-retry in v1. User taps Retry manually. Keeps the auth flow predictable and avoids the duplicate-account concern entirely. For idempotent reads (`GET /v1/me`), one retry-on-timeout might be acceptable, but defer.
- **[ASKED OF]:** Android + iOS

## Resolved

Move resolved questions here with the answer and the date. Don't delete — they become a record.
