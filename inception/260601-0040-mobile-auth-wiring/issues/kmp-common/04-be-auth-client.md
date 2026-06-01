---
type: issue
feature: mobile-auth-wiring
lane: kmp-common
status: in-progress
claimed-by: NguyenKhacPhuc
wave: 0
estimate: 60m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/mobile-auth-wiring
  - status/in-progress
  - wave/0
---

# [kmp-common] BE auth API client wraps the 4 endpoints in typed calls

**Lane:** kmp-common
**PRD section:** [[PRD#Goals]] (the BE-call goals in stories 1 + 3)
**API contract section:** [[api-contract]] (entire file)

## Why

The sign-in screen ViewModel (story 05) and the Settings ViewModel (story 06) both need to call the BE. Centralizing those calls behind one typed client means: (a) request / response shapes and error-envelope parsing live in exactly one place, (b) each call site gets a small typed result (success-with-payload, validation-error, conflict, unauthenticated, rate-limited, network-error) instead of raw HTTP plumbing, (c) authed calls automatically attach the bearer from [[01-session-token-store-interface]].

## Acceptance criteria

Foundation slice — observable through the client's public method surface against a mocked HTTP engine. No real network in tests.

- [ ] A `kmp-common` HTTP client exposes typed methods for the 4 endpoints from [[api-contract#Endpoints consumed by the mobile client]]: sign-up, sign-in, get-me, sign-out.
- [ ] Each method returns a sealed typed result that distinguishes: success (with the parsed payload), each `error.code` the BE returns for that endpoint, and a generic network/transport failure.
- [ ] Authed calls (`getMe`, `signOut`) automatically attach the bearer token from `SessionTokenStore` if present; if the store has no token, the methods short-circuit with an "unauthenticated" result without making the HTTP call.
- [ ] Request bodies are serialized to JSON matching the api-contract exactly (camelCase field names, no extras).
- [ ] Response parsing tolerates extra fields the server might add later (no breaking on unknown JSON keys).
- [ ] Validation rules from [[api-contract#Validation rules — client mirrors BE]] live in one shared helper that BOTH the sign-up route handler in story 05 AND the sign-in route handler in story 05 use; the client itself does NOT re-validate (it just submits and reports the BE's response).
- [ ] Network timeout for any call is bounded (Construction picks the value, default 10s acceptable).

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes. Likely shape: tests against `MockEngine` for Ktor client; assertions on the request shape AND the typed-result mapping per HTTP-code / error-code combo.
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64`.

## Blocked by

- nothing — independently grabbable. The `SessionTokenStore` interface from [[01-session-token-store-interface]] is needed for the bearer-attaching behavior; Construction may decide to take both stories together as one PR if that's cleaner, OR build this against a stub interface and integrate when story 01 lands.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract.

- **Likely dep:** Ktor client is already used somewhere in the host app per `undercurrent/CLAUDE.md` (`data/network` module). Reuse the existing Ktor client setup if its engine + plugins suit; otherwise scope a new HTTP module.
- **Existing pattern to mirror:** the sealed-typed-result pattern is common in the codebase — look for existing repository patterns under `core/domain` that return `Result`-ish sealed classes.
- **Watch out for:** base URL hardcoding for v1 is acceptable (see [[api-contract#Base URL]]) but consider a tiny `AuthEnvironment` config object that holds it so a future staging environment slot is a 1-line change.
- **Watch out for:** kotlinx.serialization with `ignoreUnknownKeys = true` for forward-compat on responses; the request side should be strict (BE rejects extras).

## Out of scope for this story

- Either platform's token store impl — [[../android/02-android-encrypted-session-token-store]] / [[../ios/03-ios-keychain-session-token-store]].
- The UI calling into the client — [[05-first-launch-sign-in-screen]] / [[06-settings-account-and-sign-out]].
- Retries — per [[../../open-questions#Q6]] no auto-retry in v1.
- Refresh tokens — explicitly NOT a thing on the BE side (see [[../../../260531-1733-backend-bootstrap-auth/decisions#D3]]).
