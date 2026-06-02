---
type: prd
feature: weft-ios-parity
status: approved
created: 2026-06-02
tags:
  - inception/prd
  - feature/weft-ios-parity
  - status/approved
---

# PRD: Weft iOS parity — a common, turnkey iOS mechanism

> [!success] **Status:** Approved 2026-06-02 — ready for Construction · **Driver:** SteveCastalk · **Last updated:** 2026-06-02
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

Make the Weft substrate set up and run on iOS with the **same one-call mechanism Android already has**, pushing as much wiring as possible into shared code so an iOS host app stops hand-assembling the SDK.

## Problem

Weft already *compiles* for iOS — most of it (contracts, security, MCP, the whole agent harness, tools, compose, the WebView/Html components) is genuinely shared, working code. But the **setup mechanism is Android-only in practice**:

- An Android host gets a fully-wired SDK from one call. An **iOS host has to hand-assemble it** — supplying its own stand-ins for every device capability and its own composition wiring, because the iOS side of the device-capability layer is entirely unimplemented (33 placeholder capabilities that throw if touched).
- The undercurrent host app has already written real iOS versions of a couple of those capabilities (secure credential storage, voice input) **inside the app**, where no other Weft host can reuse them. That's duplicated work waiting to happen for every future iOS host.
- Sign-in via an external provider works turnkey on Android but **has no iOS launcher**, so each iOS host wires its own.
- The debug overlay that Android developers get for free **isn't available on iOS at all**.

Net: "iOS is supported" is technically true and practically false. The cost lands on every iOS host developer, starting with undercurrent.

This feature does **not** chase full device-capability coverage (that's a deliberately deferred backlog — see [[out-of-scope]]). It makes the **mechanism** common and proves it end-to-end with a foundational set of real capabilities.

## Goals

Testable goals.

- [ ] An iOS host app obtains a fully-wired set of device capabilities from a **single setup call**, the same shape Android offers — no hand-assembly of stand-ins.
- [ ] The SDK's composition/setup is **shared across platforms** rather than duplicated per platform, so adding an iOS host doesn't mean re-deriving how the SDK is put together.
- [ ] A **foundational set of device capabilities works for real on iOS**: secure credential storage, OS permission prompts, clipboard, opening links, haptic feedback, screen/brightness control, image editing, sharing, system/device info, and voice input.
- [ ] Capabilities **not yet implemented on iOS report a clear "not available on this device" outcome** instead of crashing the agent.
- [ ] Signing in through an external provider **works turnkey on iOS**, matching the Android experience.
- [ ] The **debug overlay is available on iOS**.
- [ ] The undercurrent host app is **migrated to the common mechanism** and **deletes its in-app copies** of the now-shared capabilities — demonstrating the duplication is actually gone.
- [ ] Everything that was shareable stays **compiling on both Android and iOS** with no Android regression.

## Non-goals

- Full device-capability coverage on iOS (location, camera, calendar, contacts, vision/OCR, PDF, notifications, media picker, audio recording, bluetooth, sensors, translation, wifi, installed-apps, telephony, shortcuts, system-settings deep links, media library, volume control). Deferred — see [[out-of-scope]].
- Any new user-facing app feature in undercurrent beyond adopting the common mechanism.
- Backend work — this feature is entirely client/SDK-side.
- Android behavior changes (other than incidental refactors needed to share composition code).

## User stories

> [!note]
> The "user" for most of these stories is the **iOS host-app developer** consuming Weft, and the **person using an iOS app built on Weft**. Stories are written in terms of observable outcomes for those two, not internal class shapes.

### Story 1 — One-call iOS setup

**As an** iOS host-app developer, **I want** to obtain a fully-wired Weft SDK from a single setup call, **so that** I don't hand-assemble device capabilities and composition the way I do today.

**Acceptance criteria:**
- [ ] An iOS host gets a working capability set from one call, without supplying its own stand-ins.
- [ ] The same call, used with no custom overrides, yields an SDK that runs an agent turn on a real iOS device.

### Story 2 — Shared setup, not duplicated

**As a** maintainer, **I want** the SDK's composition to live in shared code, **so that** an iOS host and an Android host are assembled from the same mechanism rather than two parallel ones.

**Acceptance criteria:**
- [ ] Setting up the SDK on iOS and on Android goes through the same shared mechanism.
- [ ] Adding a new host platform doesn't require re-deriving how the SDK is composed.

### Story 3 — Foundational capabilities work on iOS

**As a** person using an iOS app built on Weft, **I want** the agent to actually use my device, **so that** core actions behave the same as on Android.

**Acceptance criteria:**
- [ ] Credentials the agent stores persist securely across app restarts on iOS.
- [ ] The agent can prompt for and check OS permissions on iOS.
- [ ] The agent can read and write the clipboard, open links, trigger haptic feedback, keep the screen awake / adjust brightness, edit images, share content via the system share sheet, report device/system info, and accept voice input — all on iOS.

### Story 4 — Graceful gaps

**As a** person using an iOS app built on Weft, **I want** unsupported actions to fail cleanly, **so that** the agent doesn't crash when it tries something iOS doesn't yet support.

**Acceptance criteria:**
- [ ] When the agent invokes a capability not yet implemented on iOS, it gets a clear "not available on this device" result rather than a crash.

### Story 5 — Turnkey sign-in on iOS

**As a** person using an iOS app built on Weft, **I want** to sign in to an external provider, **so that** the agent can use my connected services — the same as on Android.

**Acceptance criteria:**
- [ ] Tapping "connect" launches the provider's sign-in flow on iOS and returns to the app on completion.
- [ ] Cancelling the sign-in flow returns to the app cleanly with no connection made.

### Story 6 — Debug overlay on iOS

**As an** iOS host-app developer, **I want** the debug overlay on iOS, **so that** I can inspect agent activity the way Android developers can.

**Acceptance criteria:**
- [ ] The debug overlay opens on iOS and shows agent activity for the current session.

### Story 7 — Undercurrent adopts the mechanism

**As the** undercurrent host app, **I want** to switch to the common setup and delete my in-app capability copies, **so that** the duplication is provably gone.

**Acceptance criteria:**
- [ ] Undercurrent's iOS build uses the common one-call setup.
- [ ] Undercurrent no longer carries its own in-app copies of the capabilities now provided by the SDK.
- [ ] The iOS app behaves the same as before the migration for the capabilities it already used.

## Success metrics

- **Host hand-wiring removed:** undercurrent's iOS build composes the SDK in **one call with zero in-app capability stand-ins** (today: hand-composed with a fake capability set + 2+ in-app real impls). Measured by diff at migration.
- **Foundational capability parity:** **10/10** of the foundational capabilities (credential vault, permissions, clipboard, links, haptics, power, image-edit, sharing, system-info, voice) work on a real iOS device. Measured by manual device check + the substrate test suite.
- **No regression:** Android + iOS shared modules compile green and the substrate test suite passes with no Android behavior change. Measured by CI on the substrate PRs.

## Constraints

- **Minimum iOS deployment target: iOS 15.** (Ratified — see [[decisions]] D2.) APIs newer than iOS 15 are avoided in this feature's scope.
- **No deadline.** Waves are planned for parallelism, not speed.
- **Substrate lane = `weft/` submodule** — its own GitHub remote and PR. The host-adoption story lands in `undercurrent/`. The two are separate PRs in separate repos (see [[decisions]] D3).
- **Long-tail capabilities are explicitly deferred**, not forgotten — [[out-of-scope]] is the backlog.

## Links

- API contract: `./api-contract.md` (no BE work)
- Context: `../../CONTEXT.md`
- Issues: `./issues/`
