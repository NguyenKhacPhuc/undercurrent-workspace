---
type: out-of-scope
feature: mobile-auth-wiring
created: 2026-06-01
tags:
  - inception/out-of-scope
  - feature/mobile-auth-wiring
---

# Out of scope

> [!warning]
> Things this feature is explicitly **not** doing. Be generous. The cheapest argument to prevent is one you wrote down.

- **Editing display name or email.** BE has no `PATCH /v1/me` yet (see [[../260531-1733-backend-bootstrap-auth/out-of-scope]]). Settings shows the account read-only. When PATCH lands, a small follow-up Inception wires the editor.
- **Forgot-password / password reset.** BE doesn't expose the endpoint. No mobile UI for it; users who forget reinstall. Tracked under [[decisions#D6]].
- **Email verification.** BE accepts unverified emails; mobile shows whatever the server returns.
- **Sign in with Apple / Google / any third-party identity.** Future Inception once the BE adds OAuth.
- **Sync of conversations, personas, mini-apps, memory.** Each gets its own Inception once we have an authed identity to scope them to.
- **Migration from the (never-shipped) local `UserProfile` from [[../260531-1719-sign-in-flow/PRD]].** None of that code ever landed in production, so there's nothing on-device to migrate.
- **Offline mode / queued auth attempts.** Sign-in needs a working network. We surface that clearly when it isn't (Story 1 AC).
- **App-wide connectivity banner.** Belongs in a connectivity Inception, not this one.
- **Rate-limit countdown timer.** Per [[decisions#D5]], the 429 message is plain — no live countdown.
- **Multi-account on a single install.** One signed-in account per device.
- **"Active sessions" management screen** (list of devices, "sign out everywhere"). The BE supports multi-session; surfacing it to users is a future polish.
- **Account deletion.** No `DELETE /v1/me` on the BE yet; defer.
- **Biometric unlock** (Face ID / fingerprint to re-auth without password). Future Inception once we want the friction trade-off.
- **"Remember me" toggle** on sign-in. Sessions are always 30 days per [[../260531-1733-backend-bootstrap-auth/decisions#D3]] — toggle would be a lie.
- **Telemetry on the email value itself off-device.** We count completions, not contents.
- **Localization of validation / error copy beyond English.** Picked up by whatever the broader i18n effort is.
