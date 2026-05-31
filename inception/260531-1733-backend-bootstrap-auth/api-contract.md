---
type: api-contract
feature: backend-bootstrap-auth
created: 2026-05-31
backend-work: true
tags:
  - inception/api-contract
  - feature/backend-bootstrap-auth
---

# API contract

> [!warning] **Keystone artifact.**
> When this file has zero `TBD` markers, BE / Android / iOS can proceed in parallel.

This is the first BE Inception in the workspace. There is no existing API shape to inherit; the conventions below ARE the conventions going forward, and the mob ratifying this Inception ratifies them for future BE Inceptions.

## Conventions

- **Auth:** Bearer token. `Authorization: Bearer <session-token>`. The token is opaque to clients (no JWT decoding) — see [[decisions#D3]].
- **Base URL:** `https://undercurrent-backend-production.up.railway.app` *(pinned 2026-05-31 after Story 01 deploy; Railway-generated, custom domain deferred per Q1 resolution).*
- **Path versioning:** all endpoints live under `/v1/`. Major-version bumps move to `/v2/`. No minor-version paths.
- **Error envelope:** all non-2xx responses return
  ```json
  { "error": { "code": "<machine_code>", "message": "<human message>", "details": { ... }? } }
  ```
  `code` is a stable string; clients branch on it. `message` may change wording; clients show it. `details` is optional, error-specific, and may include validation field paths.
- **Date format:** ISO 8601 UTC, e.g. `2026-05-31T17:33:00Z`.
- **Content type:** request + response bodies are `application/json; charset=utf-8`.
- **Standard error codes used below:** `invalid_request`, `unauthenticated`, `email_already_registered`, `rate_limited`.

## Endpoints

### `GET /health` — liveness probe

**Auth:** public

**Request:** none.

**Success response (200):**

```json
{ "status": "ok" }
```

**Notes:** intentionally minimal. Used by Railway's healthcheck and the BE team's external uptime checker. No DB touch.

---

### `POST /v1/auth/sign-up` — register a new account

**Auth:** public

**Request:**

```json
{
  "displayName": "Phuc",
  "email": "phuc@example.com",
  "password": "<plain text>"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `displayName` | string | yes | Non-empty after trim, ≤ 40 chars (matches `UserProfile.displayName`). |
| `email` | string | yes | Lowercased + trimmed server-side before uniqueness check. Format rule per [[open-questions#Q2]]. |
| `password` | string | yes | Minimum length per [[open-questions#Q2]]. Driver guess: ≥ 8 chars, no max. Never logged. |

**Success response (201):**

```json
{
  "account": {
    "id": "acct.<uuid12>",
    "displayName": "Phuc",
    "email": "phuc@example.com",
    "createdAtMs": 1748707980000
  },
  "session": {
    "token": "<opaque>",
    "expiresAtMs": 1751299980000
  }
}
```

**Error responses:**

| Code | When | `error.code` |
|---|---|---|
| 400 | missing/invalid fields | `invalid_request` |
| 409 | email already has an account | `email_already_registered` |

**Notes:**
- Idempotency: not idempotent. Two simultaneous identical sign-ups race on the email uniqueness check; one wins with 201, the other gets 409.
- Pagination: n/a.

---

### `POST /v1/auth/sign-in` — exchange credentials for a session

**Auth:** public

**Request:**

```json
{
  "email": "phuc@example.com",
  "password": "<plain text>"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `email` | string | yes | Lowercased + trimmed server-side. |
| `password` | string | yes | Checked against the stored hash. Never logged. |

**Success response (200):**

```json
{
  "account": {
    "id": "acct.<uuid12>",
    "displayName": "Phuc",
    "email": "phuc@example.com",
    "createdAtMs": 1748707980000
  },
  "session": {
    "token": "<opaque>",
    "expiresAtMs": 1751299980000
  }
}
```

**Error responses:**

| Code | When | `error.code` |
|---|---|---|
| 400 | missing fields | `invalid_request` |
| 401 | wrong password OR unknown email | `unauthenticated` (same code + same `error.message` for both, to prevent enumeration) |
| 429 | rate-limited on this email | `rate_limited` (see Story 7) |

**Notes:**
- Successful sign-in does NOT invalidate other live sessions for the same account (a user can be signed in on multiple devices). See PRD Story 4 AC.

---

### `GET /v1/me` — return the signed-in account

**Auth:** required

**Request:** none.

**Success response (200):**

```json
{
  "account": {
    "id": "acct.<uuid12>",
    "displayName": "Phuc",
    "email": "phuc@example.com",
    "createdAtMs": 1748707980000
  }
}
```

**Error responses:**

| Code | When | `error.code` |
|---|---|---|
| 401 | missing / unknown / revoked / expired token | `unauthenticated` |

**Notes:**
- Idempotency: yes (pure read).
- Caching: clients should NOT cache; the BE returns no `Cache-Control` or sets `private, no-store`.

---

### `POST /v1/auth/sign-out` — invalidate the presented session

**Auth:** required (loose — see notes)

**Request:** none.

**Success response (204):** empty body.

**Error responses:** none under normal use. See notes for why.

**Notes:**
- **Always 204.** Per PRD Story 6 AC: missing / unknown / already-invalid tokens still respond 204. The endpoint is idempotent and leaks no information.
- Invalidates only the presented token, not all sessions of the account.

---

## Notes on what is NOT in this contract

- No `/v1/auth/forgot-password`, no `/v1/auth/verify-email`, no `/v1/auth/change-password`. See [[out-of-scope]].
- No `PATCH /v1/me`. Profile mirroring is a future Inception.
- No CSRF protections — the API is consumed by native mobile clients only; there is no browser surface in v1.
