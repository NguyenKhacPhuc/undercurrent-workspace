---
type: decisions
feature: backend-bootstrap-auth
created: 2026-05-31
tags:
  - inception/decisions
  - feature/backend-bootstrap-auth
---

# Decisions

> [!info]
> ADR-lite log. One entry per decision the mob made (or that the driver made and the mob ratified). The point is to prevent re-litigating the same argument in week 3.
>
> This is the workspace's first BE-touching Inception — several of the decisions below set defaults that future Inceptions will inherit.

## To ratify

### D1 — Backend lives in a new `backend/` submodule — 2026-05-31

- **Context:** Per the workspace `CLAUDE.md` "When BE arrives" guidance, the first BE-touching feature has to decide where BE code lives: new submodule, inside `weft/`, or inside `undercurrent/`.
- **Options considered:** (a) new `backend/` submodule sibling to `weft/` and `undercurrent/`; (b) inside `undercurrent/` (e.g. `undercurrent/backend/`); (c) inside `weft/`.
- **Decision:** (a) new `backend/` submodule.
- **Why:** Matches the existing two-submodule pattern (one git remote per concern, independent CI, independent release cadence). Avoids tangling BE deploys with host-app releases. Keeps Weft's identity as a substrate SDK, not a full-stack framework.
- **Consequences:**
  - Workspace `CLAUDE.md` lane table must change `backend` from "dormant — TBD" to point at `backend/<module>/` + `backend/CLAUDE.md`. Story 1 owns the edit.
  - The workspace's pinned commits grow to 3.
  - Inception artifacts for future BE features keep landing under `inception/<feature>/issues/backend/` (workspace root), as today.

### D2 — Email + password is the v1 auth model (no OAuth, no magic link, no email verification) — 2026-05-31

- **Context:** v1 auth has many viable shapes — Sign in with Apple / Google, magic link, OTP, email+password, anonymous-then-link. Each comes with different friction, infra cost, and security profile.
- **Options considered:** Sign in with Apple + Google (OAuth); email + one-time code (magic link / OTP); email + password; trust-the-typed-email (no challenge).
- **Decision:** Email + password. No third-party SDK on day one. Adjacent flows (forgot-password, email verification, change-password) are out-of-scope for v1; see [[out-of-scope]].
- **Why:** Simplest mental model for the BE team (no third-party token validation, no email-sending dependency). Easy to add OAuth or magic link later — they coexist on the same account record.
- **Consequences:**
  - Users who forget their password are locked out until forgot-password ships. Mitigation: scope is closed-beta-ish; the lockout risk is bounded.
  - We must hash passwords properly (argon2id default — Construction confirms). PRD Story 3 AC enforces "stored as a salted hash, never as plain text."
  - The local sign-in screen built in [[../260531-1719-sign-in-flow/]] does not currently collect a password. The mobile-wiring follow-up Inception will reshape it.

### D3 — Sessions are opaque, server-stored, long-lived (30 days), no refresh tokens in v1 — 2026-05-31

- **Context:** Session token shape choices: JWT (stateless, easy to scale, hard to revoke); opaque server-stored (easy to revoke, requires DB lookup per request); short-lived JWT + refresh token (most complex, best of both).
- **Options considered:** opaque server-stored; signed JWT; short-lived JWT + refresh token.
- **Decision:** Opaque, stored in a `sessions` table, validated by DB lookup on every authenticated request. Default TTL 30 days from issuance, no refresh token, no sliding-window. Sign-out is a row deletion / `revoked_at` set.
- **Why:** Sign-out is in scope (PRD Story 6); opaque tokens make revocation a one-line operation. We don't have a horizontal-scale problem in v1 — a single Railway dyno + a DB lookup per request is fine. JWT can come later if a real perf reason emerges.
- **Consequences:**
  - Sessions table grows linearly with sign-ins; need a janitor (cron / on-startup sweep) to purge expired rows eventually. Tracked in [[out-of-scope]] for v1 — not a problem at low volume.
  - Token rotation on long-lived sessions is not implemented; a leaked token works for up to 30 days. Acceptable for v1; revisit if we ever surface "active sessions" management to users.

### D4 — This Inception is BE-only; mobile-client integration is a separate follow-up — 2026-05-31

- **Context:** The BE could be specced alongside the mobile-side wiring (one Inception, three lanes: `backend`, `kmp-common`, possibly `android`/`ios` for any platform-specific concern). Driver chose to keep this Inception BE-only.
- **Options considered:** BE-only this Inception; BE + minimal mobile wiring; BE + full mobile sign-in rework.
- **Decision:** BE-only.
- **Why:** Lets the BE team iterate freely on the endpoint shape without coordinating with the mobile lanes. The api-contract here becomes the public surface mobile binds against in a follow-up. Lower coordination cost during a high-uncertainty bootstrap.
- **Consequences:**
  - The existing [[../260531-1719-sign-in-flow]] feature does not call this BE; both ship into the app independently. The mobile-wiring Inception is named in [[out-of-scope]].
  - This Inception's `issues/` tree is entirely `backend/` — no other lane directories.

### D5 — Tech stack: Ktor on JVM; storage: Railway Postgres; deploy: Railway from GitHub `main` — 2026-05-31

- **Context:** First-ever BE; tech-stack decisions echo through everything. Driver picked.
- **Options considered:** Stack — Ktor (Kotlin), Spring Boot, FastAPI, Node, Go. Deploy — Railway, Fly.io, Render, AWS. DB — Postgres, MySQL, SQLite-on-server.
- **Decision:** Ktor (Kotlin/JVM), deployed on Railway with auto-deploy from the `backend/` submodule's `main` branch. Storage: Railway-managed Postgres (DB choice rolls out of Railway being chosen; Postgres is the default Railway add-on).
- **Why:** Ktor keeps the entire workspace in Kotlin — driver and reviewers don't pay a language-context-switch tax. Plays well with future "share types between BE and KMP client" if we ever go there. Railway picked for lowest-friction first deploy and managed Postgres in one click; the workspace already has a `railway-deployment` skill.
- **Consequences:**
  - Cold-start latency from JVM is worse than a Node/Go BE; acceptable for an auth API where p99 < 500ms is fine.
  - We commit to Kotlin + Gradle inside `backend/` — Construction sets up the build skeleton.
  - Migration of off Railway later, if needed, requires a Dockerfile (Railpack should already build one).

### D6 — Sign-in rate limiting: 10 failures / email / 15-minute window — 2026-05-31

- **Context:** Story 7 / Story 9 needed concrete thresholds. Per-IP rate limiting was already out of scope (Railway proxy trust setup).
- **Options considered:** 5/15min, 10/15min, 20/1h.
- **Decision:** 10 failures per email per 15-minute rolling window. 11th+ attempt responds 429 `rate_limited` until the window passes with no failures. Any successful sign-in clears the counter immediately.
- **Why:** Comfortable margin for a confused user typing variants of a remembered password without giving a grinder useful throughput. 15 min recovery window is short enough that a real user is rarely fully blocked for long.
- **Consequences:** A targeted DoS *can* lock a specific known email out of sign-in for 15-min windows by spraying wrong passwords. Acceptable risk at v1 traffic; revisit if it becomes a reported abuse.

### D7 — Secrets management is Railway env vars only for v1 — 2026-05-31

- **Context:** Q6 weighed Railway-only vs `.env.example` checked in vs a dedicated vault.
- **Options considered:** Railway env vars only; Railway + `.env.example`; Doppler / Infisical / 1Password CLI.
- **Decision:** Railway env vars only. No `.env.example`. No vault.
- **Why:** Total secret count for v1 (≤ 5: `DATABASE_URL`, a session-token signing/pepper seed if used, Railway-injected service tokens) fits comfortably in Railway's env-var UI. A vault is operational overhead with no payoff at this surface.
- **Consequences:** Onboarding a second BE engineer requires manually pointing them at Railway env vars (no self-serve `.env.example` to copy). Trigger to revisit: ≥ 10 secrets OR second BE engineer onboarding.

### D8 — Observability is Railway logs + UptimeRobot on /health; no Sentry / APM in v1 — 2026-05-31

- **Context:** PRD lists "service availability" as a success metric but didn't pin how it's measured. Q7 asked how observability lands.
- **Options considered:** Railway logs + UptimeRobot only; same + Sentry for error reporting; defer everything.
- **Decision:** Railway's built-in log viewer for structured BE logs. UptimeRobot free tier hitting `/health` every 5 minutes — that's the data source for the PRD's "service availability" metric. No Sentry / no APM.
- **Why:** Free, fast to set up, sufficient for closed-beta-ish traffic. Sentry's value grows with traffic and team size; not yet worth the integration.
- **Consequences:** Unhandled exceptions and 5xx errors are visible only by scrolling Railway logs — no aggregated error reporting. Acceptable while traffic is low; obvious upgrade target once it isn't.

### D9 — No CI workflow on `backend/` submodule in v1 — 2026-05-31

- **Context:** Q8 asked whether GitHub Actions should run tests on push / PR. Driver chose to skip in v1, contrary to the recommended baseline.
- **Options considered:** GH Actions test on push + PR (recommended); none; recommended + lint.
- **Decision:** No CI. The BE team runs `./gradlew test` locally before pushing; Railway's auto-deploy from `main` is the only automated check.
- **Why:** Cheaper to start. The driver is comfortable with the regression risk at v1 scale and would rather add CI when there's a second contributor or a regression actually happens.
- **Consequences:** A broken `main` will deploy. Mitigations: only one contributor at v1; local-test discipline; the `/health` endpoint will visibly fail in UptimeRobot if a regression breaks startup. Add a CI workflow when the team grows OR after the first prod regression — whichever comes first.

### D10 — Test-stack for DB code: Testcontainers Postgres (added during Construction of Story 02) — 2026-05-31

- **Context:** Story 02's migration runner needs DB-roundtrip tests. Choices weighed during Construction: Testcontainers, H2 in Postgres-compat mode, or skip-and-rely-on-deploy.
- **Options considered:** Testcontainers (real Postgres in Docker); H2 with `MODE=PostgreSQL` (~80% compat); skip DB tests entirely.
- **Decision:** Testcontainers. `org.postgresql:postgresql` 42.7.4 added as runtime dep; `org.testcontainers:postgresql` + `:junit-jupiter` 1.20.4 as test-only deps.
- **Why:** Most accurate signal — tests exercise the same SQL semantics production sees. The driver is comfortable with Docker as a local dependency. H2's compat gap would have bitten us by Story 04 (sessions table) when we want `INSERT ... ON CONFLICT` or `RETURNING`.
- **Consequences:** Anyone running the BE tests needs Docker installed and running. Without CI (per D9), that's only the driver's machine; acceptable. Tests are slower than H2 would be (~5-10s for Postgres container spin-up), but tolerable for a small suite.

---

*(Future entries appended as the mob ratifies / overrides / adds.)*
