---
type: issue
feature: ios-agent-bringup
lane: kmp-common
status: in-progress
claimed-by: SteveCastalk
wave: 0
estimate: 60m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ios-agent-bringup
  - status/ready
  - wave/0
---

# [kmp-common] Share the substrate-backed history, memory, trace & usage repositories

**Lane:** kmp-common (`undercurrent/`)
**PRD section:** [[PRD]] → Story 1 — Shared data layer
**API contract section:** n/a (no BE)

## Why

The repositories that read conversations, memory, traces, and usage from the SDK live in `androidMain` only because the SDK's runtime used to be Android-bound. The runtime is shared now, so these repositories can be shared too — one implementation for both platforms instead of an Android impl + an iOS stub.

## Acceptance criteria

- [ ] The conversation-history, memory, trace, and usage repositories that read from the SDK are shared across Android and iOS (no longer Android-only).
- [ ] On Android, conversation history, memory, traces, and usage behave exactly as before.
- [ ] The shared repositories compile for both Android and iOS.

## Blocked by

- nothing — independently grabbable

## Hints (non-binding)

> [!tip]
> Read `undercurrent/CLAUDE.md`. These are pure delegations over the substrate's stores — lifting the class, not rewriting it.

- **Likely files affected:** `undercurrent/core/domain/src/androidMain/…/Weft{Memory,Trace,Usage,ConversationStore}Repository.kt` → `…/commonMain/…`. Confirm against actual code.
- **Substrate pieces consumed:** the runtime's conversation / memory / trace / usage stores (all commonMain now).
- **Watch out for:** the existing iOS `IosConversationStoreRepository` (local-SQLite) — decide whether it folds into the shared substrate-backed one or stays; don't regress Android.
- **Verify (from `undercurrent/`):** `./gradlew :core:domain:test :core:domain:compileDebugKotlinAndroid :core:domain:compileKotlinIosSimulatorArm64`

## Out of scope for this story

- The iOS DI wiring that binds these (that's [[06-ios-adopt-shared-layer]]).
- Key vault and agent host (separate stories).
