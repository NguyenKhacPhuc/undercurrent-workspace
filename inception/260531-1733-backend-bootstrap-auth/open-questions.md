---
type: open-questions
feature: backend-bootstrap-auth
created: 2026-05-31
tags:
  - inception/open-questions
  - feature/backend-bootstrap-auth
---

# Open questions

> [!success] **All questions resolved — 2026-05-31.**
> Definition-of-done for this artifact is met. Resolutions are below in "Resolved"; substantive new commitments are recorded in [[decisions]] (D6 – D9).
>
> Mob still needs to formally review [[decisions]] before Construction starts on any story.

## Open

*(none)*

## Resolved

### Q1 — Public BE URL / hostname strategy — 2026-05-31

- **Answer:** Ship behind Railway's generated URL (e.g. `*.up.railway.app`) for v1. Move to a custom domain only when sign-up volume or branding justifies the DNS + cert work.
- **By:** driver
- **Promoted to:** [[api-contract#Conventions]] — Base URL is now pinned: `https://undercurrent-backend-production.up.railway.app` (set after Story 01's first deploy, 2026-05-31).

### Q2 — Exact validation rules for email, password, displayName — 2026-05-31

- **Answer:** Adopt driver-guess defaults.
  - `displayName`: non-empty after trim; ≤ 40 chars; unicode (incl. emoji) allowed.
  - `email`: must contain `@` with a `.` after it and no spaces; case-insensitive uniqueness (server lowercases before storing/matching).
  - `password`: ≥ 8 chars; no maximum; no required character classes.
- **By:** driver
- **Promoted to:** [[issues/backend/05-sign-up-endpoint]] and [[issues/backend/06-sign-in-endpoint]] AC. The pointer those stories use ("see Q2 — driver-guess values OK") now reads as ratified.

### Q3 — Rate-limit thresholds for Story 7 — 2026-05-31

- **Answer:** 10 failed sign-in attempts per email per 15-minute window. After the 10th failure, the 11th and subsequent attempts respond 429 for the rest of the window. Counter resets on any successful sign-in (per Story 9 AC). Per-IP rate limiting remains out of scope.
- **By:** driver
- **Promoted to:** [[decisions#D6]]; [[issues/backend/09-sign-in-rate-limiting]] AC.

### Q4 — Session TTL + refresh strategy — 2026-05-31

- **Answer:** 30 days from issuance, no refresh token. Already captured in [[decisions#D3]]; this just confirms.
- **By:** driver
- **Promoted to:** no new decision needed (D3 already covers).

### Q5 — Does this Inception re-open prior sign-in-flow decisions? — 2026-05-31

- **Answer:** No. Leave [[../260531-1719-sign-in-flow/]] as-is. A separate mobile-wiring Inception (after Construction here) will reshape the sign-in screen to add a password field and bind to the BE.
- **By:** driver
- **Promoted to:** the existing PRD `Constraints` section already names this; no new decision needed. [[../260531-1719-sign-in-flow/decisions#D2]] and that Inception's Q5 stay as written.

### Q6 — Secrets management beyond Railway env vars — 2026-05-31

- **Answer:** Railway env vars only for v1. No vault, no `.env.example`. Revisit at ≥ 10 secrets or when a second BE engineer onboards.
- **By:** driver
- **Promoted to:** [[decisions#D7]].

### Q7 — Observability: logging / error reporting / metrics — 2026-05-31

- **Answer:** Railway's built-in log viewer for structured BE logs. UptimeRobot (free tier) hitting `/health` every 5 minutes, supplying the "service availability" PRD success metric. No Sentry / no APM in v1.
- **By:** driver
- **Promoted to:** [[decisions#D8]]. PRD success metric updated to name UptimeRobot specifically.

### Q8 — CI on the new `backend/` submodule — 2026-05-31

- **Answer:** **No CI workflow in v1.** Trust local `./gradlew test` before push. Railway's auto-deploy from `main` is the only automated check. *(Driver chose this over the recommended GH Actions setup; explicit trade-off — cheaper to start, easier to regress.)*
- **By:** driver
- **Promoted to:** [[decisions#D9]]; [[issues/backend/01-ktor-deploys-to-railway-with-health]] hint.
