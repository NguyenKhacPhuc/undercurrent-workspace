---
type: api-contract
feature: backend-driven-prompt
created: 2026-06-19
backend-work: true
tags:
  - inception/api-contract
  - feature/backend-driven-prompt
---

# API contract

> [!warning] **Keystone artifact.**
> When this file has zero `TBD` markers, BE / kmp-common / Android / iOS can proceed in parallel. The client lanes work against this contract with a mock until BE lands.

## Conventions

Reuses the existing backend surface (auth feature, `inception/260531-1733-backend-bootstrap-auth/`).

- **Auth:** opaque bearer token — `Authorization: Bearer <session-token>`. The client fetch happens after sign-in, using the existing session token.
- **Base URL:** `https://undercurrent-backend-production.up.railway.app`.
- **Success envelope:** `BaseResponse<T> { success, data, message, code }`; client unwraps `.data`.
- **Error envelope:** `{ "code": "<machine_code>", "message": "<human message>", "details": object? }`.
- **Date format:** epoch milliseconds.

## The prompt config

A single, global record (one row) — the current base prompt the assistant runs on.

| Field | Type | Notes |
|---|---|---|
| `preamble` | string | The full base prompt text (app intro + behavioral / anti-hallucination rules). Authoritative; replaces the previously compiled-in constant. |
| `revision` | string | An opaque marker that changes whenever `preamble` changes (lets a client cheaply tell "is what I have current?"). |
| `updatedAtMs` | number | epoch ms of the last change. |

## Endpoints

### `GET /v1/prompt-config` — fetch the current base prompt

**Auth:** required (bearer). Fetched at startup, after sign-in, with the existing session token.

**Success response (200):**

```json
{
  "success": true,
  "data": {
    "preamble": "You are Undercurrent…\n\nNever narrate a tool call you don't make…",
    "revision": "rev.000004",
    "updatedAtMs": 1750339200000
  },
  "code": "ok"
}
```

**Error responses:**

| Code | When | `code` |
|---|---|---|
| 401 | missing/expired token | `unauthenticated` |
| 503 | config not yet seeded / temporarily unavailable | `unavailable` |

**Notes:**
- Idempotency: yes (read-only).
- Caching: client caches the last successful payload locally and prefers it when offline. A client may send the cached `revision` so the server can answer cheaply if unchanged (a not-modified response is acceptable — BE's choice).
- This is the **client-facing** endpoint and is fully specified — no TBDs.

---

### `PUT /v1/prompt-config` — replace the current base prompt (operator)

**Auth:** operator-only. The exact operator-authorization mechanism is a parked decision — see [[open-questions#Q3]] (the mob defers the mechanism; the endpoint's shape is fixed).

**Request:**

```json
{ "preamble": "You are Undercurrent… (new text)" }
```

| Field | Type | Required | Notes |
|---|---|---|---|
| preamble | string | yes | non-empty; the new full base prompt |

**Success response (200):**

```json
{ "success": true, "data": { "revision": "rev.000005", "updatedAtMs": 1750342800000 }, "code": "ok" }
```

**Error responses:**

| Code | When | `code` |
|---|---|---|
| 400 | empty / missing preamble | `invalid_request` |
| 401 / 403 | not an authorized operator | `unauthenticated` / `forbidden` |

**Notes:**
- Each successful update bumps `revision` and `updatedAtMs`.
- A safety check on what may be applied (e.g. reject an empty or obviously-broken prompt, given there is no client fallback) is in scope to discuss — see [[open-questions#Q2]].
- Seeding: the config is initialized with the current built-in prompt text so cut-over is behavior-neutral (handled in the serve story, not a separate endpoint).
