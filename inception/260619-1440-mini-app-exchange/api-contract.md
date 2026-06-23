---
type: api-contract
feature: mini-app-exchange
created: 2026-06-19
backend-work: true
tags:
  - inception/api-contract
  - feature/mini-app-exchange
---

# API contract

> [!warning] **Keystone artifact.**
> When this file has zero `TBD` markers, BE / kmp-common / Android / iOS can proceed in parallel. The mobile lanes work against this contract with a local mock until BE lands.

## Conventions

Reuses the existing backend surface introduced by the auth feature (`inception/260531-1733-backend-bootstrap-auth/api-contract.md`).

- **Auth:** opaque bearer token — `Authorization: Bearer <session-token>`. Same token issued by sign-in / sign-up.
- **Base URL:** `https://undercurrent-backend-production.up.railway.app` — hardcoded for v1 (same as auth feature).
- **Success envelope:** all 2xx JSON wrapped in `BaseResponse<T> { success, data, message, code }`; the client unwraps `.data`.
- **Error envelope:** `{ "code": "<machine_code>", "message": "<human message>", "details": object? }`.
- **Date format:** epoch milliseconds (matches existing `*Ms` fields across the app).
- **IDs:** share records use a short, URL-safe identifier `share.<base62-10>` (short enough to encode in a compact QR; opaque, unguessable).

## The share bundle

A **share bundle** is the portable representation of an HTML mini-app. It carries everything a recipient needs to preview, clamp, and install — and deliberately nothing about the sharer's local state (saved app state is per-device and is not shared).

| Field | Type | Notes |
|---|---|---|
| `name` | string | Display name. ≤ 60 chars. |
| `emoji` | string | Single emoji icon. |
| `description` | string? | Optional author blurb shown on preview. ≤ 200 chars. |
| `html` | string | The complete self-contained HTML document (inline JS/CSS, no remote scripts). |
| `declaredScopes` | string[] | The capabilities the mini-app declares it needs (e.g. `["http_fetch","store_get"]`). |

> [!note] The recipient's app re-clamps `declaredScopes` against its own offerable actions at install time. The bundle is a *request*, never a grant.

## Endpoints

### `POST /v1/mini-apps/share` — create a share link for a mini-app

**Auth:** required (must be signed in to share).

**Request:**

```json
{
  "name": "Water tracker",
  "emoji": "💧",
  "description": "Tap to log a glass; see today's count.",
  "html": "<!doctype html>…",
  "declaredScopes": ["store_get", "store_set"]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| name | string | yes | ≤ 60 chars |
| emoji | string | yes | single emoji |
| description | string | no | ≤ 200 chars |
| html | string | yes | ≤ 256 KB (see Notes — size cap) |
| declaredScopes | string[] | yes | may be empty; values are free-form capability names |

**Success response (201):**

```json
{
  "success": true,
  "data": {
    "shareId": "share.7Kd2mPq9Xa",
    "url": "https://undercurrent-backend-production.up.railway.app/s/share.7Kd2mPq9Xa",
    "deepLink": "undercurrent://install/share.7Kd2mPq9Xa",
    "createdAtMs": 1750339200000
  },
  "code": "ok"
}
```

| Field | Type | Notes |
|---|---|---|
| shareId | string | `share.<base62-10>` |
| url | string | Human-shareable https link (share-sheet / messaging). |
| deepLink | string | Custom-scheme link the QR encodes; opens the app preview directly. |
| createdAtMs | number | epoch ms |

**Error responses:**

| Code | When | `code` |
|---|---|---|
| 400 | missing/oversized html, missing name | `invalid_request` |
| 401 | missing/expired token | `unauthenticated` |
| 413 | html exceeds size cap | `payload_too_large` |

**Notes:**
- Idempotency: not idempotent — each POST mints a new share record. (Re-sharing the same mini-app twice yields two links; acceptable for v1.)
- The record stores the bundle plus the owner `accountId` and the sharer's `displayName` snapshot (so preview attribution survives even if the account later changes its name).
- Size cap: `html` ≤ **256 KB**. See [[decisions#D5]].

---

### `GET /v1/mini-apps/share/{shareId}` — fetch a shared mini-app for preview/install

**Auth:** public (no account needed to receive).

**Success response (200):**

```json
{
  "success": true,
  "data": {
    "shareId": "share.7Kd2mPq9Xa",
    "name": "Water tracker",
    "emoji": "💧",
    "description": "Tap to log a glass; see today's count.",
    "html": "<!doctype html>…",
    "declaredScopes": ["store_get", "store_set"],
    "authorName": "Sam",
    "sharedAtMs": 1750339200000
  },
  "code": "ok"
}
```

| Field | Type | Notes |
|---|---|---|
| shareId | string | echoes the requested id |
| name / emoji / description / html / declaredScopes | — | the share bundle |
| authorName | string | the sharer's display-name snapshot, shown on the preview |
| sharedAtMs | number | epoch ms the share was created |

**Error responses:**

| Code | When | `code` |
|---|---|---|
| 404 | unknown id **or** the share was revoked | `not_found` |

> [!note] Revoked and never-existed are intentionally indistinguishable (both 404) — the recipient just sees "no longer available". No information leak about whether an id once existed.

**Notes:**
- Idempotency: yes (read-only).
- No auth → no rate-coupling to an account; apply a coarse per-IP rate limit (reuse the existing limiter; exact number is BE's call).

---

### `DELETE /v1/mini-apps/share/{shareId}` — stop sharing (revoke)

**Auth:** required; caller must be the owner of the share record.

**Success response (200):**

```json
{ "success": true, "data": { "shareId": "share.7Kd2mPq9Xa", "revoked": true }, "code": "ok" }
```

**Error responses:**

| Code | When | `code` |
|---|---|---|
| 401 | missing/expired token | `unauthenticated` |
| 403 | caller is signed in but not the owner | `forbidden` |
| 404 | unknown id (or already revoked) | `not_found` |

**Notes:**
- Idempotency: yes — revoking an already-revoked share returns 404 (`not_found`); the end state (link dead) is the same either way. (Alternatively BE may treat a second revoke as 200; either is acceptable — pick one and document in the BE story.)
- Revocation is a soft state on the record (a `revokedAtMs`), not a hard delete — keeps the 404-on-revoke behavior cheap and auditable.
- Already-installed recipients are unaffected: their copy lives entirely on their device.
