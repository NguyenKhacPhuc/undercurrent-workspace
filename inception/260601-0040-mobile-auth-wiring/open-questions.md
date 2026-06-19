---
type: open-questions
feature: mobile-auth-wiring
created: 2026-06-01
tags:
  - inception/open-questions
  - feature/mobile-auth-wiring
---

# Open questions

> [!success] **All questions resolved — 2026-06-19.**
> Definition-of-done for this artifact is met. All six driver-guesses were ratified by the driver (SteveCastalk); resolutions are below in "Resolved".
>
> All 6 stories are merged (`status: done`). Only on-device smoke testing remains, and it is deferred (no hardware available 2026-06-19).

## Open

*(none)*

## Resolved

Move resolved questions here with the answer and the date. Don't delete — they become a record.

### Q1 — Client mirrors BE validation rules exactly? — 2026-06-19

- **Answer:** Yes, mirror the BE rules exactly. Encode email (loose format), password (≥8), and displayName (non-empty trimmed, ≤40) in one shared `AuthValidation` helper in `kmp-common` so they live in exactly one place; any future change is a one-line edit. This lets the client fail fast with inline errors before any submit.
- **By:** Driver (SteveCastalk)

### Q2 — UX form for register-vs-sign-in mode switch on the same screen? — 2026-06-19

- **Answer:** A simple toggle at the top of the form: "Sign In | Register". Default = Sign In (most users on a re-install / second-device install). Tapping the toggle swaps modes; common fields (email, password) preserve their values; register adds the displayName field below.
- **By:** Driver (SteveCastalk)

### Q3 — What does "wipe local token + route to sign-in screen" look like *during* an authed user flow? — 2026-06-19

- **Answer:** The current screen is interrupted, a brief toast says "You were signed out — please sign in again", and the sign-in screen appears with the email field pre-filled from the last known account. In-flight transient state (typed messages, partial tool calls) is lost. Acceptable for v1; we are NOT building an in-memory queue.
- **By:** Driver (SteveCastalk)

### Q4 — Mobile telemetry for auth events? — 2026-06-19

- **Answer:** Reuse the existing trace harness if it can emit non-LLM-turn events; otherwise add a tiny `AuthTelemetry` interface in `kmp-common` with a noop default + a future swap-in for whatever analytics target ships next. Counting only (no email content, no token content).
- **By:** Driver (SteveCastalk)

### Q5 — First-launch race: what if the user has a stored token but it was revoked / expired server-side? — 2026-06-19

- **Answer:** Do a `GET /v1/me` validation call on every cold launch with a stored token. While in flight: show a brief splash / spinner over the would-be home surface (no flash). On 200: render home. On 401: wipe + route to sign-in. On network error: optimistically render home (the next authed call will eventually fail and route correctly).
- **By:** Driver (SteveCastalk)

### Q6 — Should the BE client retry transient network errors automatically? — 2026-06-19

- **Answer:** No auto-retry in v1. User taps Retry manually. Keeps the auth flow predictable and avoids the duplicate-account concern entirely. For idempotent reads (`GET /v1/me`), one retry-on-timeout might be acceptable, but is deferred.
- **By:** Driver (SteveCastalk)
