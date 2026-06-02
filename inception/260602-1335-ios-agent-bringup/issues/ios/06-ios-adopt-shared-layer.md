---
type: issue
feature: ios-agent-bringup
lane: ios
status: in-progress
claimed-by: SteveCastalk
wave: 2
estimate: 60m
blocked-by:
  - "[[01-share-history-usage-repos]]"
  - "[[02-share-secure-key-repo]]"
  - "[[04-ios-sdk-at-launch]]"
  - "[[05-share-chat-streaming]]"
tags:
  - inception/issue
  - lane/ios
  - feature/ios-agent-bringup
  - status/ready
  - wave/2
---

# [iOS] The agent answers on iOS — adopt the shared layer

**Lane:** iOS (`undercurrent/`)
**PRD section:** [[PRD]] → Story 3 — The agent answers on iOS
**API contract section:** n/a (no BE)

## Why

This is the payoff: swap the iOS app's stub repositories for the shared, substrate-backed ones, so the chat screen goes live. After this, an iOS user can send a message and watch the agent reply.

## Acceptance criteria

- [ ] On iOS, sending a message streams the agent's reply token-by-token.
- [ ] The reply persists in conversation history and is present after restart.
- [ ] Memory, traces, and usage screens show real data from the SDK (not empty).
- [ ] The chat screen is reachable once a provider key is present (no longer routed back to key-paste).
- [ ] The iOS app no longer carries the agent-path stubs (chat, memory, traces, usage) for the in-scope capabilities.

## Blocked by

- [[04-ios-sdk-at-launch]] — needs the runtime.
- [[01-share-history-usage-repos]], [[02-share-secure-key-repo]], [[05-share-chat-streaming]] — the shared impls this wires in.

## Hints (non-binding)

- **Likely files affected:** `undercurrent/composeApp/src/iosMain/…/IosKoinModule.kt` — replace `StubChatRepository`, `StubMemoryStoreRepository`, `StubTraceStoreRepository`, `StubUsageRepository`, and the standalone key repo with the shared substrate-backed bindings; delete `StubAgentEngine` / `StubChatRepository` if nothing else references them.
- **Watch out for:** the chat screen's `isReady` gate — it must flip true once a key is present so navigation reaches chat.
- **Verify (from `undercurrent/`):** `./gradlew :composeApp:compileKotlinIosSimulatorArm64` + iOS app build; manual e2e (send "hello").

## Out of scope for this story

- Provider/agent switching + permission dialogs + boot-flow polish ([[07-ios-agent-lifecycle]]).
- OAuth integrations ([[08-ios-integrations-signin]]).
- Model catalog, key validation, voice — deferred ([[out-of-scope]]).
