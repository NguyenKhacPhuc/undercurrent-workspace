---
type: api-contract
feature: mobile-auth-wiring
created: 2026-06-01
backend-work: false
tags:
  - inception/api-contract
  - feature/mobile-auth-wiring
---

# API contract

> [!success] **No backend changes.** The mobile-wiring feature consumes the existing BE surface. The canonical contract is [[../260531-1733-backend-bootstrap-auth/api-contract]]; this file just summarizes what the mobile side calls and what shapes it must handle.

## Endpoints consumed by the mobile client

| Endpoint | Auth | Mobile use |
|---|---|---|
| `POST /v1/auth/sign-up` | public | Story 1 — register mode submit |
| `POST /v1/auth/sign-in` | public | Story 1 — sign-in mode submit |
| `GET /v1/me` | bearer | Story 3 — Settings → Account section render |
| `POST /v1/auth/sign-out` | bearer (loose) | Story 3 — Sign Out tap (200 with empty success envelope) |
| `GET /health` | public | Optional — used as a connectivity probe? Currently no; we treat any non-200 from the auth endpoints as the connectivity signal. |

## Base URL

`https://undercurrent-backend-production.up.railway.app` — same value pinned in [[../260531-1733-backend-bootstrap-auth/api-contract#Conventions]]. Mobile clients hardcode this for v1 (no per-environment selection). When we add staging, switch to a `BuildConfig` constant.

## Bearer-token format

Opaque base64url string ~43 chars. Treated as a credential — never logged, never displayed, stored only in secure platform storage.

## Error envelope (what the client must parse on 4xx)

```json
{ "code": "<machine_code>", "message": "<human message>", "details": object? }
```

Decoded into mobile's `BaseErrorResponse(code: String, message: String, details: Map<String, String>? = null)`.

Error codes the client branches on:

| `code` | When | Client UX |
|---|---|---|
| `invalid_request` | 400 on sign-up / sign-in | Inline error per field if `details` is populated; else show `message`. |
| `email_already_registered` | 409 on sign-up | Inline message + "Switch to sign-in" shortcut with email pre-filled. |
| `unauthenticated` | 401 on sign-in OR on any authed call | On sign-in: show "Invalid email or password". On authed call: wipe local token + route to sign-in screen. |
| `rate_limited` | 429 on sign-in | Show "Too many failed attempts. Try again later." inline. |

## Success shapes (subset the client cares about)

All 2xx JSON responses are wrapped in `BaseResponse<T> { success, data, message, code }` — the client uses `safeApiCall { ... }` which unwraps `.data` to the payload below.

### `AuthResponse` (200 / 201 from sign-up / sign-in)

```json
{
  "success": true,
  "data": {
    "account": { "id": "acct.<uuid12>", "displayName": "...", "email": "...", "createdAtMs": 1748707980000 },
    "session": { "token": "<opaque>", "expiresAtMs": 1751299980000 }
  },
  "message": null,
  "code": null
}
```

### `GET /v1/me` 200

```json
{
  "success": true,
  "data": { "account": { "id": "...", "displayName": "...", "email": "...", "createdAtMs": ... } },
  "message": null,
  "code": null
}
```

### `POST /v1/auth/sign-out` 200

```json
{ "success": true, "data": null, "message": null, "code": null }
```

## Validation rules — client mirrors BE

Per BE [[../260531-1733-backend-bootstrap-auth/decisions#D2]] + Q2 resolution:

- `displayName`: non-empty after trim; ≤ 40 chars; unicode incl. emoji allowed.
- `email`: must contain `@` followed somewhere by `.` and no whitespace; case-insensitive uniqueness server-side (client just normalizes for display).
- `password`: ≥ 8 chars; no maximum; no required character classes.

Client enforces these BEFORE submit so the user gets fast inline feedback. Server is the ultimate authority — any divergence is a client bug.
