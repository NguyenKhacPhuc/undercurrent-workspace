---
type: decisions
feature: weft-ios-parity
created: 2026-06-02
tags:
  - inception/decisions
  - feature/weft-ios-parity
---

# Decisions

> [!success]
> **Ratified by the mob 2026-06-02.** D1–D5 below are approved; this feature's
> Inception is complete and Construction may begin. ADR-lite log retained as the
> record of why the spec looks the way it does.

---

### D1 — Scope is "common mechanism", not full capability coverage — 2026-06-02

- **Context:** The iOS device-capability layer has 33 unimplemented capabilities. Full parity is ~3 weeks of Construction. The driver's intent was "make it a common and usable mechanism as much as Android, common as much as possible" — emphasis on the *mechanism* being common, not on grinding every capability.
- **Options considered:** (a) Full parity — all 33 capabilities; (b) MVP critical path — ~16 headline capabilities; (c) Foundation only — the turnkey setup mechanism + lift the host's existing impls + the no-permission quick wins.
- **Decision:** Option (c), **plus** the shared-composition refactor and the iOS sign-in launcher (the two pieces that make the mechanism genuinely *common* rather than just present).
- **Why:** The expensive, repeated cost today is the *hand-assembly* every iOS host pays, not any single missing capability. Fixing the mechanism once unblocks the long tail to land incrementally as ordinary follow-ups. Proving it with 10 real capabilities + a graceful-fallback path is enough to demonstrate end-to-end.
- **Consequences:** ~20 device capabilities stay deferred (see [[out-of-scope]]). They are now *cheap* to add — each becomes a single capability slice against an existing mechanism — but they are not in this feature.

### D2 — Minimum iOS deployment target is iOS 15 — 2026-06-02

- **Context:** Several iOS APIs are version-gated; the floor decides which are usable.
- **Options considered:** iOS 15, iOS 16.
- **Decision:** **iOS 15.**
- **Why:** iOS 15 covers everything this feature touches (system sign-in flow, system share sheet, secure storage, permissions) while keeping the widest device reach. Nothing in the foundational set needs iOS 16+.
- **Consequences:** Any future capability needing iOS 16/17 APIs must guard by version. Out of scope here.

### D3 — Substrate work and host adoption are separate PRs in separate repos — 2026-06-02

- **Context:** The bulk of the work is in `weft/`; the host-adoption story is in `undercurrent/`. They live in different submodules with different GitHub remotes.
- **Options considered:** One combined change vs. two coordinated PRs.
- **Decision:** All capability + mechanism work lands as PR(s) in `weft/`. The undercurrent migration lands as its own PR in `undercurrent/`, after the substrate ships and is bumped in.
- **Why:** Matches the workspace rule — substrate changes land in `weft/` and are pulled into the host reactively. Keeps each PR reviewable against its own repo's checks.
- **Consequences:** The host-adoption story (ios lane) is cross-repo-blocked on the substrate setup story; reflected in its `blocked-by` and in [[_index]]'s final wave.

### D4 — Undercurrent's in-app iOS capability impls move down into the SDK — 2026-06-02

- **Context:** Undercurrent already wrote real iOS versions of secure credential storage and voice input *inside the app*. Every future iOS host would otherwise rewrite them.
- **Options considered:** Leave them in the host; copy them into the SDK; move them into the SDK and delete the host copies.
- **Decision:** **Move them into the SDK and delete the host copies** as part of the adoption story.
- **Why:** "Common as much as possible" means the host stops owning substrate-shaped code. A copy that isn't deleted is duplication that drifts.
- **Consequences:** The adoption story must verify behavior is unchanged after the host loses its local copies.

### D5 — Permission usage-description strings are the host's responsibility — 2026-06-02

- **Context:** iOS kills an app at permission-request time if the matching usage-description string is missing. The SDK provides the capability; the string is app config. Resolves [[open-questions]] Q1.
- **Options considered:** Host owns the strings (SDK documents what each capability needs); SDK ships default strings the host overrides.
- **Decision:** **The host owns the strings.** The SDK documents the required string per permission-gated capability but ships none.
- **Why:** Usage strings are user-facing, app-specific, and need per-host localization — baking wording into the substrate would leak app identity into shared code (against the architectural rule) and fight localization.
- **Consequences:** [[01-host-adopt-substrate-capabilities]] adds the microphone + speech strings. Every future iOS host must add strings for any permission-gated capability it adopts; the SDK's per-capability docs are the checklist.

### D6 — iOS native tests in weft use `kotlin.test` + kotest matchers; entitlement-gated capabilities verify their write paths manually — 2026-06-02

- **Context:** Story 02 ([[02-ios-credential-vault]]) needed the first-ever native (iOS) test in weft. Two walls surfaced during Construction: (a) kotest's native spec runner needs the `io.kotest.multiplatform` compiler plugin, which is version-locked and risky against weft's Kotlin; (b) a bare Kotlin/Native `iosSimulatorArm64Test` binary has no app-host entitlement, so the data-protection Keychain rejects writes with `errSecNotAvailable (-25291)`.
- **Options considered:** kotest-engine + compiler plugin vs `kotlin.test` + kotest matchers; for the keychain writes: pursue an entitled test host, add a prod seam for an in-memory fake, or `@Ignore` the writes + manual on-device verification.
- **Decision:** Native tests use **`kotlin.test` for discovery + kotest matchers (`shouldBe`) for assertions** — no kotest compiler plugin. Capability **write paths that require an app-host entitlement are `@Ignore`d with an explanatory note and verified manually**; the runnable read/not-found paths stay in CI.
- **Why:** Gets real, low-risk CI coverage today without a fragile compiler-plugin bump, and without polluting production code with a test-only seam or a wrong keychain flag. The pattern generalizes to every future iOS capability (PR [#2](https://github.com/NguyenKhacPhuc/android-harness/pull/2)).
- **Consequences:** A new `iosTest` source set exists in `:os-bridge` (reusable). Future entitlement-gated iOS capabilities (e.g. voice/[[11-ios-voice-input]]) follow the same split. A future infra story could add an entitled iOS test host to graduate the `@Ignore`d write tests into CI.
