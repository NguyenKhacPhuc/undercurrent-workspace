---
type: issue
feature: ios-agent-bringup
lane: kmp-common
status: done
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/15
merged-at: 2026-06-02T09:34:31Z
claimed-by: SteveCastalk
wave: 1
estimate: 90m
blocked-by:
  - "[[03-share-agent-builder]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ios-agent-bringup
  - status/done
  - wave/1
---

# [kmp-common] Share the streaming chat repository

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 3 — The agent answers on iOS
**API contract section:** n/a (no BE)

## Why

The chat repository is the heart of the agent experience: it takes a user message, drives the agent, and streams back the reply (text deltas, tool-start/done/fail, completion) while persisting the conversation. It lives in the chat feature's `androidMain` today. Move it to shared code so iOS gets the same streaming pipeline — this is what makes the agent answer on iOS.

## Acceptance criteria

- [ ] Sending a message drives the agent and streams the reply (token deltas + tool progress + completion) through one shared repository on both platforms.
- [ ] The conversation lifecycle (new chat, resume, select, delete, regenerate-last, switch agent) works through the shared repository.
- [ ] A failed turn streams a clear failure rather than hanging.
- [ ] Android chat behaves exactly as before.
- [ ] The shared repository compiles for both Android and iOS.

## Blocked by

- [[03-share-agent-builder]] — the streaming repo drives the agent the builder produces.

## Hints (non-binding)

- **Likely files affected:** lift `undercurrent/feature/chat/src/androidMain/…/internal/WeftChatRepository.kt` → `…/commonMain/…`.
- **Substrate pieces consumed:** the agent's state flow + effects flow, mapped to the host's chat-chunk stream.
- **Watch out for:** this is the highest-risk lift — careful state/effect → chunk mapping. The Android impl is the exact reference; preserve its behavior. Update `undercurrent/CLAUDE.md`'s "feature modules must not depend on Weft" note ([[open-questions]] Q2).
- **Verify (from `undercurrent/`):** `./gradlew :feature:chat:test :feature:chat:compileDebugKotlinAndroid :feature:chat:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- iOS DI wiring that binds this as the real `ChatRepository` ([[06-ios-adopt-shared-layer]]).
- App-lifecycle orchestration ([[07-ios-agent-lifecycle]]).
