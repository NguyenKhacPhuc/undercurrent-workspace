---
type: feature-index
feature: backend-bootstrap-auth
status: draft
created: 2026-05-31
tags:
  - inception/index
  - feature/backend-bootstrap-auth
  - status/draft
---

# Backend bootstrap + email/password auth — feature index

> [!info] **Status:** Draft / awaiting mob review
> Generated from Inception phase. Update this file as issues land.

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] *(4 endpoints + `/health`, 0 TBDs — base URL pinned after Story 01 deploy)*
- Decisions: [[decisions]] *(9 to ratify — D1–D5 from drafting, D6–D9 from Q-resolution pass)*
- Open questions: [[open-questions]] *(0 open — all 8 resolved 2026-05-31)*
- Out of scope: [[out-of-scope]]
- Project-wide context: [[../../CONTEXT]]
- Sibling Inception (client side): the earlier sign-in-flow Inception

---

## Parallel work plan

Issues are grouped into **waves** by dependency depth. All issues in a wave can be picked up simultaneously by different devs.

> [!tip]
> Compute wave for each issue: 0 if `blocked-by` is empty, otherwise `1 + max(wave of each blocker)`.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-ktor-deploys-to-railway-with-health]] | 90m | backend |

### 🟡 Wave 1 — unlocked once Wave 0 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[02-postgres-and-migrations]] | 60m | backend | `01` |

### 🟠 Wave 2 — unlocked once Wave 1 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[03-account-record-persisted]] | 45m | backend | `02` |
| [[04-session-issued-validated-revoked]] | 60m | backend | `02` |

### 🔵 Wave 3 — fan-out: 4 endpoint stories in parallel

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[05-sign-up-endpoint]] | 60m | backend | `03`, `04` |
| [[06-sign-in-endpoint]] | 60m | backend | `03`, `04` |
| [[07-get-current-user-endpoint]] | 30m | backend | `03`, `04` |
| [[08-sign-out-endpoint]] | 30m | backend | `04` |

### 🟣 Wave 4 — final hardening

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[09-sign-in-rate-limiting]] | 75m | backend | `06` |

---

> [!warning] **Parallelization sanity check**
> Waves 0 and 1 are single-issue by design — bootstrap is inherently serial: there must be a deployed app before a DB can attach to it, and a DB before any persistence story can land. Wave 2 fans to 2 (account + session land in parallel since both only need the DB); Wave 3 fans to 4 (all four endpoint handlers can be written concurrently by different devs once the foundations exist). Wave 4 is a single hardening pass that depends on sign-in being live.
>
> Total focused work ≈ 8–9 hours across 9 stories. With one dev: roughly two days. With two devs from Wave 3 onwards: about a day and a half.

---

## All issues

```
issues/
└── backend/
    ├── 01-ktor-deploys-to-railway-with-health.md
    ├── 02-postgres-and-migrations.md
    ├── 03-account-record-persisted.md
    ├── 04-session-issued-validated-revoked.md
    ├── 05-sign-up-endpoint.md
    ├── 06-sign-in-endpoint.md
    ├── 07-get-current-user-endpoint.md
    ├── 08-sign-out-endpoint.md
    └── 09-sign-in-rate-limiting.md
```

No `kmp-common/`, `android/`, `ios/`, or `substrate/` directories — BE-only this Inception per [[decisions#D4]]. Mobile-side wiring is a follow-up Inception.

> [!note] **How to update this index as work lands**
> When an issue's status changes, update its frontmatter (`status: ready` → `in-progress` → `done`). Use Obsidian's search `path:issues tag:#status/ready` to see what's grabbable.

---

## Definition of done (whole feature)

- [x] All 9 issues have status `done` in their frontmatter. *(All merged by 2026-05-31.)*
- [x] [[open-questions]] has zero unresolved items. *(All 8 resolved 2026-05-31.)*
- [x] [[PRD]] has at least one success metric. *(three — uptime, sign-up→authed latency P95, rate-limit trip count.)*
- [x] [[api-contract]] has zero `TBD` markers. *(Base URL pinned 2026-05-31.)*
- [x] End-to-end: from any computer with curl, a fresh sign-up → `/v1/me` → sign-out → `/v1/me` returns 200 then 401 sequence works against the deployed URL. *(Smoked 2026-06-01 — all 7 probes match api-contract. Required PR #10 `fix: Db.toJdbcUrl parses libpq-style user:pass@` to clear the post-merge deploy crash.)*
- [x] Workspace `CLAUDE.md` lane table reflects the now-live `backend` lane. *(Done in story 01.)*
