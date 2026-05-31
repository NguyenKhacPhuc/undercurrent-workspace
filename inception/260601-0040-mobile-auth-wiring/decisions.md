---
type: decisions
feature: mobile-auth-wiring
created: 2026-06-01
tags:
  - inception/decisions
  - feature/mobile-auth-wiring
---

# Decisions

> [!info]
> ADR-lite log. One entry per decision the mob made (or that the driver made and the mob ratified). The point is to prevent re-litigating the same argument in week 3.

## To ratify

### D1 — Mobile-auth is a new Inception, not a revision of the superseded sign-in-flow — 2026-06-01

- **Context:** The `backend-bootstrap-auth` shipped end-to-end on 2026-06-01, inverting the premise of the earlier sign-in-flow Inception ("No backend"). The path forward had to choose between revising the old Inception in place or starting a new one.
- **Options considered:** (a) revise in place; (b) supersede + start new; (c) audit-only / defer.
- **Decision:** (b). The old folder is read-only, banner-marked superseded, stories flipped to `status: superseded`. This Inception lives at `260601-0040-mobile-auth-wiring/`.
- **Why:** The old Inception's D1–D4 were defensible under "BE dormant"; rewriting them erases the historical context. Also matches [[../260531-1733-backend-bootstrap-auth/decisions#D4]] exactly: "a follow-up mobile-wiring Inception will reshape the sign-in screen."
- **Consequences:** No code from the old Inception ever landed (none of its stories were claimed), so there's no on-device profile or token to migrate. Validation rules + the email + display-name conventions still echo what the old Inception spec'd, because the BE adopted them.

### D2 — Token storage: Keychain on iOS, EncryptedSharedPreferences on Android — 2026-06-01

- **Context:** The 30-day bearer is a high-value secret. The old Inception's D4 said "100% commonMain, DataStore-Preferences for everything." That stance does NOT survive once we have an actual auth credential to store.
- **Options considered:** Keychain + EncryptedSharedPreferences (per-platform secure); plain DataStore-Preferences (commonMain, insecure); a KMP-shared encryption layer (`korlibs-crypto` or hand-rolled — pre-cooked devices keys ourselves; janky).
- **Decision:** Per-platform secure store. KMP-shared `SessionTokenStore` interface in commonMain; `KeychainSessionTokenStore` in iosMain; `EncryptedSharedPreferencesSessionTokenStore` in androidMain.
- **Why:** Plain DataStore-Preferences puts the bearer in a world-readable XML file on Android (and a readable file on iOS); a malicious app or extracted backup gets a live 30-day session. That's a worse posture than the BE's password hashing. The OS-provided secure stores are the standard mobile pattern and handle key rotation + device-binding for us. Per-platform impls are the right shape — the interface stays in commonMain so ViewModels never see platform code.
- **Consequences:** Re-opens the old D4. This Inception's lane tree has `kmp-common/`, `android/`, `ios/` — not commonMain-only. The Keychain entry survives uninstall on iOS (a known UX wart per Apple's behavior); we accept it because the user can sign out cleanly via Settings. On Android, EncryptedSharedPreferences ties the encryption key to the device's keystore alias; uninstall + reinstall clears it cleanly. Adds `androidx.security:security-crypto` as a new androidMain dep; iosMain uses the platform Keychain Services API directly (no new dep).

### D3 — Profile state model: BE is source of truth; client lazy-fetches `/v1/me` — 2026-06-01

- **Context:** When the client needs to render the user's display name + email (Settings, future agent greeting, etc.), where does it get them?
- **Options considered:** Lazy fetch `/v1/me` on every consumer; persisted read-only cache refreshed on sign-in + opportunistically; resurrect the old `UserProfile` local store.
- **Decision:** Lazy fetch. No persisted mirror in v1.
- **Why:** Simplest. There's no `PATCH /v1/me` yet, so the only way the profile changes is sign-up time — when we already get the full `AccountDto` in the response. Lazy-fetch from Settings is one round-trip per Settings open; acceptable since identity isn't shown anywhere else in v1.
- **Consequences:** Settings shows a loading placeholder while `/v1/me` is in flight. If the network is bad, Settings shows "Couldn't load account" with Retry (NOT a sign-out — only a 401 wipes the token). When PATCH /v1/me eventually lands, this decision likely gets revisited toward an Account cache + invalidation pattern.

### D4 — Sign-out from Settings is in v1, with a BE revoke call — 2026-06-01

- **Context:** Sign-out is a Story 6 endpoint on the BE side that's already live. Question is whether mobile v1 includes the affordance and how cleanly it acts.
- **Options considered:** Yes + BE revoke; yes + local-only (skip BE revoke); no — defer entirely.
- **Decision:** Yes, with the BE revoke call. The Sign Out tap calls `POST /v1/auth/sign-out` (best-effort; succeeds on 204 OR on network error — the local wipe still happens), then wipes the local token, then routes to the sign-in screen.
- **Why:** Shared-device safety. Without sign-out, a user who hands their phone over still has a live session. The BE revoke is "best-effort" because we'd rather get the user signed out locally than block them on a flaky network — the trade-off is a token that's revoked locally but still live server-side until natural 30-day expiry. Acceptable.
- **Consequences:** If a network error happens during sign-out, we silently complete the local wipe + route to sign-in. The user is signed out from their perspective. If the BE call eventually succeeds in the background — fine; if not, the BE session expires naturally. Add a future story for "active sessions list" if/when this becomes a felt problem.

### D5 — 429 UX is a plain message; no countdown timer; no fake "Forgot password" link — 2026-06-01

- **Context:** When the BE returns 429 `rate_limited` (10 failures in 15-min window), the user is locked out for up to 15 minutes. What does the UI say?
- **Options considered:** Plain message; same + client-side countdown estimate; same + a "Forgot password?" link.
- **Decision:** Plain message ("Too many failed attempts. Try again later."). No countdown. No "Forgot password" link.
- **Why:** Honest. The BE doesn't return a `retry-after` and we don't keep client-side counter state; a client-side estimate would drift. A "Forgot password" link is a lie we can't fulfill (the BE doesn't expose recovery — see [[out-of-scope]]).
- **Consequences:** A locked-out user has no in-app remedy. They wait, or — if they've genuinely forgotten their password — they reinstall the app (losing nothing, since v1 has no persisted on-device data tied to the account). The lack of a recovery path is documented as a known gap; revisit when BE adds forgot-password.

### D6 — Forgot password is out of scope for mobile v1 — 2026-06-01

- **Context:** Should mobile ship a forgot-password UI when the BE doesn't expose the endpoint yet?
- **Options considered:** No link, no UI; stub link to a "how to recover" docs page; pause this Inception until BE catches up.
- **Decision:** No link, no UI. Track as a known gap in `out-of-scope.md`.
- **Why:** Stub links create future cleanup work and damage trust. Pausing the Inception holds up everything we DO want to ship.
- **Consequences:** Users who forget their password are stuck reinstalling. Acceptable since v1 has nothing meaningful to lose on reinstall. Triggers a future BE Inception to add `POST /v1/auth/forgot-password`; when that lands, a small mobile follow-up wires the UI.

### D7 — Onboarding order: sign-in / register first, then provider / API-key — 2026-06-01

- **Context:** Resolves the earlier sign-in-flow Inception's Q6. Today fresh installs see a provider-picker + API-key paste flow. Where does sign-in go?
- **Options considered:** Sign-in first then provider; provider first then sign-in; one combined carousel; defer to mob.
- **Decision:** Sign-in first, then the existing provider/key step. Two distinct screens; no merging.
- **Why:** Identity-before-config is the mental model the BE-backed world assumes. The two are independently meaningful (the user IS someone before they have a provider configured).
- **Consequences:** Story 1's "block until sign-in completes" stops at the home surface — the existing provider/key onboarding then runs as it does today. Construction owns the splice into ScreenRouter's start-destination logic.

---

*(Future entries appended as the mob ratifies / overrides / adds.)*
