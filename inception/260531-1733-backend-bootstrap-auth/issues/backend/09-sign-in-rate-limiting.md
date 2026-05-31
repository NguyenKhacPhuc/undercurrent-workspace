---
type: issue
feature: backend-bootstrap-auth
lane: backend
status: done
claimed-by: NguyenKhacPhuc
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent-backend/pull/9
merged-at: 2026-05-31T16:54:44Z
wave: 4
estimate: 75m
blocked-by:
  - "[[06-sign-in-endpoint]]"
tags:
  - inception/issue
  - lane/backend
  - feature/backend-bootstrap-auth
  - status/done
  - wave/4
---

# [backend] Repeated failed sign-in attempts on the same email are throttled

**Lane:** backend
**PRD section:** [[PRD#Story 7 — Sign-in throttles brute-force attempts]]
**API contract section:** [[api-contract#`POST /v1/auth/sign-in` — exchange credentials for a session]] (the `429` row)

## Why

Without throttling, an attacker can grind passwords against the live sign-in endpoint at request-rate. Argon2id slows them down but doesn't stop them. This story plugs the actual brake.

## Acceptance criteria

- [ ] After 10 failed sign-in attempts against the same email within a 15-minute rolling window (per [[../../decisions#D6]]), the 11th and subsequent attempts on that email respond 429 with `error.code = "rate_limited"`, regardless of whether the next attempt would have been correct.
- [ ] Throttling on one email does not affect sign-in for any other email.
- [ ] After a quiet window with no failed attempts, throttling clears automatically.
- [ ] A successful sign-in resets the failure count for that email so a user who finally types the right password isn't punished after recovering.
- [ ] The 429 response is timing-similar to the normal 401 (i.e. the rate-limit check shouldn't open a new timing oracle) — but no extra goldplating required; matching the normal response shape is enough.
- [ ] The thresholds (10 failures, 15-minute window) live in one named place so a future tuning pass can adjust them without code archaeology.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes.
- The lane's standard build/test commands pass with no regressions.

## Blocked by

- [[06-sign-in-endpoint]] — needs the call-site hook the sign-in handler stubs out.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Storage:** Construction picks — could be in-memory per-instance (acceptable on a single-dyno Railway deploy), could be a `sign_in_failures` DB table (survives restarts, multi-dyno safe). Trade-off is operational, not user-facing.
- **Watch out for:** per-IP rate limiting is explicitly out of scope. Don't add it just because it's tempting — the trust-the-proxy-IP work is a real rabbit hole on Railway.
- **Watch out for:** the success-resets-counter rule. Implementing it wrong (e.g. "counter only clears after window") means a user who types it correctly on attempt 11 still gets locked out — bad UX, not in spec.

## Out of scope for this story

- Per-IP rate limiting.
- Rate limiting on other endpoints (sign-up, `/v1/me`, sign-out). v1 only throttles sign-in.
- CAPTCHA fallback after lockout. See [[../../out-of-scope]].
