---
type: issue
feature: ios-agent-bringup
lane: kmp-common
status: done
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/14
merged-at: 2026-06-02T08:40:02Z
claimed-by: SteveCastalk
wave: 0
estimate: 75m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ios-agent-bringup
  - status/done
  - wave/0
---

# [kmp-common] Share the agent-building machinery

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 3 — The agent answers on iOS (prerequisite)
**API contract section:** n/a (no BE)

## Why

Building an agent — resolving the active provider's key into a credential, selecting the agent declaration, and constructing the runnable agent — lives in the chat feature's `androidMain` because the SDK used to be Android-only. The SDK is shared now, so this machinery can move to shared code, ready for iOS to use.

## Acceptance criteria

- [ ] The agent holder + agent-builder (provider key → credential → runnable agent) are shared across platforms.
- [ ] Building an agent for a given provider + key yields a runnable agent on both platforms.
- [ ] Android agent-building behaves exactly as before.
- [ ] The shared machinery compiles for both Android and iOS.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

- **Likely files affected:** lift `undercurrent/feature/chat/src/androidMain/…/agent/{AgentSlot,WeftAgentFactory}.kt` → `…/commonMain/…`. The chat feature's commonMain may now depend on the KMP substrate ([[decisions]] D2; see [[open-questions]] Q2).
- **Substrate piece consumed:** the runtime's `buildAgent(...)` + credential-provider types (commonMain).
- **Watch out for:** `undercurrent/CLAUDE.md` forbids feature modules depending on Weft — that rule predates the KMP substrate; relax it for KMP-published substrate modules (note it here).
- **Verify (from `undercurrent/`):** `./gradlew :feature:chat:test :feature:chat:compileDebugKotlinAndroid :feature:chat:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- The streaming chat repository that drives the agent (that's [[05-share-chat-streaming]]).
- iOS DI wiring ([[06-ios-adopt-shared-layer]]).
