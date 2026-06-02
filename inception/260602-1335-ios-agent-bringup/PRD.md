---
type: prd
feature: ios-agent-bringup
status: draft
created: 2026-06-02
tags:
  - inception/prd
  - feature/ios-agent-bringup
  - status/draft
---

# PRD: iOS agent bring-up — undercurrent on the real substrate

> [!info] **Status:** Draft / awaiting mob review · **Driver:** SteveCastalk · **Last updated:** 2026-06-02
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items. Downstream of the merged **weft-ios-parity** substrate feature (`inception/260602-0149-weft-ios-parity/`).

## One-line intent

Turn undercurrent's iOS app from a "coming-soon" shell into a **working agent** by wiring the real Weft substrate — now that the substrate ships KMP-on-iOS with a turnkey runtime factory.

## Problem

undercurrent's iOS host is a stub. Its `StubAgentEngine` literally documents the reason: *"ships without agent capabilities — coming-soon until (a) Weft itself migrates to KMP."* `IosKoinModule` wires **~10 stub repositories** (chat, memory, traces, usage, model-catalog, key-validation, OAuth, UI-bridge, …). The iOS user can sign in and paste a key but **cannot chat with the agent** — the chat screen routes back to key-paste because `ChatRepository.isReady` is false.

The blocker named in that stub — "Weft migrates to KMP" — **is now done** (weft-ios-parity, merged 2026-06-02): the substrate runs on iOS and exposes a turnkey `WeftRuntime.create(WeftPlatform(), …)` factory plus a secure key vault and an OAuth launcher. This feature consumes that to bring the agent to life on iOS.

A second, compounding win: the host's Weft-backed repository layer currently lives in `androidMain` **only because `WeftRuntime` used to be Android-bound**. Now that `WeftRuntime` is commonMain, that layer can be **lifted to shared code** — Android and iOS run the same impls, and iOS just adds DI wiring.

## Goals

Testable goals.

- [ ] An iOS user can **send a message and get a streaming agent reply** — the chat screen is live, not routed back to key-paste.
- [ ] Conversation **history, memory, traces, and usage** on iOS are backed by the real substrate stores (not empty stubs).
- [ ] Pasted provider **keys persist securely** on iOS through the substrate's key vault and are used by the agent.
- [ ] The iOS app **switches providers / agents** and **rebuilds the agent** correctly, and surfaces permission-denied failures as a dialog (parity with Android's app behavior).
- [ ] A user can **connect an integration via OAuth** on iOS (the substrate's iOS sign-in launcher).
- [ ] The Weft-backed repository + agent-host layer is **shared in commonMain** — Android and iOS run the same impls, with no Android behavior regression.
- [ ] Everything shared **compiles on both Android and iOS**.

## Non-goals

- **Live model catalog** + **key validation ("Connect" ping)** on iOS — both need Koog's LLM clients, which are JVM/Android-only. iOS keeps the synthetic-catalog + no-validation stubs for now. See [[out-of-scope]].
- **Voice input** on iOS — blocked by the AVAudioSession Kotlin/Native binding gap (weft-ios-parity story 11, deferred). The mic CTA stays hidden on iOS. See [[out-of-scope]].
- **Backend / sync changes** — none. The agent consumes the substrate + external LLM providers; OAuth consumes external provider endpoints.
- **New user-facing features** beyond reaching Android parity for the agent experience.

## User stories

> [!note]
> The "user" is the **iOS app user** (can now chat with the agent) and the **host maintainer** (who gets a shared, de-duplicated Weft-backed layer). Stories are written as observable outcomes for those two.

### Story 1 — Shared data layer

**As a** maintainer, **I want** the substrate-backed history/memory/trace/usage repositories to live in shared code, **so that** iOS and Android run one implementation instead of two.

**Acceptance criteria:**
- [ ] The repositories that read conversations, memory, traces, and usage from the SDK are shared across platforms.
- [ ] Android behaves exactly as before.

### Story 2 — Shared secure-key storage

**As a** maintainer, **I want** the provider-key repository shared, **so that** iOS stores keys through the substrate's secure vault the same way Android does.

**Acceptance criteria:**
- [ ] Saving, reading, and clearing a provider key goes through the substrate's secure key vault on both platforms.
- [ ] A key saved on iOS survives an app restart.

### Story 3 — The agent answers on iOS

**As an** iOS user, **I want** to send a message and watch the agent reply, **so that** the app is actually usable.

**Acceptance criteria:**
- [ ] Sending a message streams the agent's reply token-by-token on iOS.
- [ ] The reply persists in conversation history and is there after restart.
- [ ] A failed turn surfaces a clear error rather than hanging or crashing.
- [ ] The chat screen is reachable (no longer routed back to key-paste once a key is present).

### Story 4 — App lifecycle parity

**As an** iOS user, **I want** provider/agent switching and clear failures, **so that** the app behaves like a finished product.

**Acceptance criteria:**
- [ ] Switching provider or agent rebuilds the agent and the next reply uses the new selection.
- [ ] A tool that fails on a missing permission surfaces a dialog with an "Open Settings" action (not an inline error bubble).
- [ ] The launch flow (sign-in → onboarding → key-paste-if-needed → chat) lands the user in the right place.

### Story 5 — Connect an integration on iOS

**As an** iOS user, **I want** to connect an external integration, **so that** the agent can use it.

**Acceptance criteria:**
- [ ] Tapping "connect" launches the provider sign-in on iOS and returns to the app on success, with the integration active.
- [ ] Cancelling returns cleanly with no connection made.

## Success metrics

- **Agent works on iOS:** send "hello" → streamed reply, on a real device. Measured by manual e2e.
- **Stub count drops:** iOS `IosKoinModule` carries **0 agent-path stubs** for the in-scope capabilities (chat, memory, traces, usage, key vault, OAuth) — was ~8. Measured by diff.
- **Shared layer:** the Weft-backed repos + agent host live in commonMain; Android + iOS compile green with no Android behavior change. Measured by CI.

## Constraints

- **Substrate is a dependency, already merged** — this feature consumes weft-ios-parity (on `weft` main). The workspace's `weft` submodule must be at/after that bump.
- **undercurrent repo hygiene:** at Inception time the host was on a dirty feature branch (`refactor/extract-auth-strings-to-resources`). Construction must branch from `undercurrent` `origin/main` and coordinate — see [[open-questions]] Q1.
- **iOS deployment target: iOS 15** (inherited from the substrate, [[CONTEXT]]).
- **Koog is JVM/Android-only** — model-catalog + key-validation stay stubbed on iOS until Koog goes KMP. Voice stays blocked on the AVAudioSession binding gap.

## Links

- API contract: `./api-contract.md` (no BE work)
- Context: `../../CONTEXT.md`
- Substrate feature it consumes: `../260602-0149-weft-ios-parity/`
- Issues: `./issues/`
