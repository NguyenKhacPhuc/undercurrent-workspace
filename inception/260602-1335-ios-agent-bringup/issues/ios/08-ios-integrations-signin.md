---
type: issue
feature: ios-agent-bringup
lane: ios
status: done
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/17
merged-at: 2026-06-02T09:46:29Z
claimed-by: SteveCastalk
wave: 1
estimate: 60m
blocked-by:
  - "[[02-share-secure-key-repo]]"
  - "[[04-ios-sdk-at-launch]]"
tags:
  - inception/issue
  - lane/ios
  - feature/ios-agent-bringup
  - status/done
  - wave/1
---

# [iOS] Connect an integration via sign-in on iOS

**Lane:** iOS (`undercurrent/`)
**PRD section:** [[PRD]] → Story 5 — Connect an integration on iOS
**API contract section:** n/a (consumes external provider OAuth via the substrate launcher)

## Why

Integrations (the OAuth-gated external systems the agent can call) are stubbed on iOS — the connect button returns "cancelled." The substrate now ships an iOS sign-in launcher, so the host can offer real integration sign-in on iOS, matching Android.

## Acceptance criteria

- [ ] Tapping "connect" on an integration launches the provider's sign-in on iOS and returns to the app on success, with the integration marked active.
- [ ] Cancelling the sign-in returns cleanly with no connection made.
- [ ] The issued tokens are stored securely and the connection survives an app restart.

## Blocked by

- [[04-ios-sdk-at-launch]] — the runtime hosts the OAuth/token plumbing.
- [[02-share-secure-key-repo]] — secure storage for the issued tokens.

## Hints (non-binding)

- **Likely files affected:** the iOS OAuth repository binding in `undercurrent/composeApp/src/iosMain/…/IosKoinModule.kt` — replace `StubOAuthRepository` with a real one over the substrate's iOS sign-in launcher; mirror the Android OAuth repository.
- **Substrate piece consumed:** the SDK's iOS provider sign-in launcher + OAuth token store (shipped in weft-ios-parity).
- **Watch out for:** the host must register the redirect URL scheme the substrate launcher returns through; integrations also typically need a process restart to pick up the new MCP server (mirror Android).
- **Verify (from `undercurrent/`):** `./gradlew :composeApp:compileKotlinIosSimulatorArm64` + iOS app build; manual connect flow.

## Out of scope for this story

- New integrations beyond the existing catalog.
- Model catalog / key validation / voice — deferred ([[out-of-scope]]).
