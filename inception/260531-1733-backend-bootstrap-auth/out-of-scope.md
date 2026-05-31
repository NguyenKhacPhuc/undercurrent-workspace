---
type: out-of-scope
feature: backend-bootstrap-auth
created: 2026-05-31
tags:
  - inception/out-of-scope
  - feature/backend-bootstrap-auth
---

# Out of scope

> [!warning]
> Things this feature is explicitly **not** doing. Be generous. The cheapest argument to prevent is one you wrote down.

- **Forgot-password / password reset.** Users who forget their password are locked out until this ships. Tracked as a follow-up; see [[decisions#D2]].
- **Email verification on sign-up.** Sign-up trusts the typed email. Spam / bot signups remain possible.
- **Change password while signed in.** Settings has no "change my password" affordance until a follow-up Inception.
- **Mobile-client integration of these endpoints.** The mobile sign-in screen does not call the BE in this release; that's a separate Inception. Driver decision; see [[decisions#D4]].
- **Sign in with Apple / Google / any third-party identity.** Email + password only in v1. The schema and endpoint shape DO leave room to add third-party identities to the same account record later, but no work is done now.
- **Sync of conversations, personas, mini-apps, memory, or any other on-device data.** Future Inceptions, each on its own.
- **Profile edit (display name / email) from the BE side.** No `PATCH /v1/me`. Edits happen only locally per [[../260531-1719-sign-in-flow/]]; cross-device propagation comes with a future sync Inception.
- **Multi-account on a single install.** v1 BE supports a single signed-in account per session token; the client only persists one at a time.
- **Account deletion / wipe.** No `DELETE /v1/me`. Compliance (GDPR-style "delete my data") will land in a separate Inception once we have actual users.
- **Per-IP rate limiting.** Story 7 covers per-email rate limiting only. Adding per-IP needs proxy-header trust setup behind Railway, which we are NOT doing in v1.
- **CAPTCHA or any human-verification on sign-up.** Acceptable risk at zero traffic; revisit when bot signups become observable.
- **Admin tooling.** No admin endpoints, no admin UI, no admin role. The BE team uses Railway's DB console for any manual fixups.
- **Sessions table janitor / expired-row purge job.** Sessions are written but expired rows are NOT swept in v1. Tracked as a follow-up before sign-up volume gets high; see [[decisions#D3]].
- **Internationalization of BE error messages.** All error envelopes return English. Client should not surface them verbatim; it should branch on `error.code`.
- **CORS / browser support.** API is consumed by native mobile only. No CORS headers; no browser-targeted dev story.
- **WebSockets, streaming, or any long-lived connection.** All endpoints are short request/response over HTTPS.
