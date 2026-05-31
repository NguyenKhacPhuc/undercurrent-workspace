---
type: issue
feature: mobile-auth-wiring
lane: kmp-common
status: ready
wave: 2
estimate: 60m
blocked-by:
  - "[[05-first-launch-sign-in-screen]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mobile-auth-wiring
  - status/ready
  - wave/2
---

# [kmp-common] Settings shows the current account and a Sign Out affordance

**Lane:** kmp-common
**PRD section:** [[PRD#Story 3 — Settings shows the current account, plus Sign Out]]
**API contract section:** [[api-contract#`GET /v1/me`]] + [[api-contract#`POST /v1/auth/sign-out`]]

## Why

After this lands, signed-in users can see who they're signed in as and sign out cleanly. Before this lands, the app's only sign-out path is reinstall — which is a bad shared-device story.

## Acceptance criteria

- [ ] Settings has an "Account" section near the top showing the user's display name and email.
- [ ] On entering Settings, the Account section triggers a `getMe` call via [[04-be-auth-client]]; while in flight it shows a loading placeholder (no flash of stale content).
- [ ] On `getMe` success, the section renders the returned display name + email.
- [ ] On a network-error result from `getMe`, the section shows "Couldn't load account" with a Retry action; the user is NOT signed out — only a 401 from the BE has that effect.
- [ ] On an `unauthenticated` result from `getMe`, the local token is wiped via [[01-session-token-store-interface]] and the user is routed back to the sign-in screen (consistent with Story 2 AC).
- [ ] A "Sign Out" affordance sits below the Account section.
- [ ] Tapping Sign Out calls `signOut` via the BE client (best-effort — treat both `204` AND network-error as success for the local wipe), wipes the local token, and routes the user back to the sign-in screen.
- [ ] The Sign Out tap shows a brief confirmation (toast / inline) so the user knows what happened.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes. Likely shape: `SettingsAccountViewModelStateTest` in commonTest against the same fake client + token store from story 05.
- The lane's standard build/test commands pass with no regressions. Same commands as story 05.

## Blocked by

- [[05-first-launch-sign-in-screen]] — semantically, sign-out has to route somewhere that exists; shipping this before 05 would create a broken state on tap.

(Transitively also depends on `01` + `04` since story 05 does, plus needs the same surfaces.)

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Likely location:** existing Settings feature module — `feature/settings/` or whichever module owns Settings today. Find the row list and append an Account section above other rows.
- **Existing pattern to mirror:** whichever Settings row currently performs a mutate-and-persist action (e.g. theme palette via `ThemePrefs`) — copy the basic shape, just replace the action with the sign-out call.
- **Watch out for:** the "best-effort signOut" interpretation — if the BE call fails with a non-204 non-network error (e.g. 5xx, but the endpoint is documented to ALWAYS return 204 — see [[api-contract#`POST /v1/auth/sign-out`]]), still complete the local wipe. The user being signed out locally is the desired state regardless of server response.

## Out of scope for this story

- Editing display name or email — no `PATCH /v1/me`; see [[../../out-of-scope]].
- "Active sessions" management UI — see [[../../out-of-scope]].
- "Sign out everywhere" affordance — see [[../../out-of-scope]].
- Account deletion — see [[../../out-of-scope]].
