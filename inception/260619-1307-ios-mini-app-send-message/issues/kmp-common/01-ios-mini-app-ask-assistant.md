---
type: issue
feature: ios-mini-app-send-message
lane: kmp-common
status: done
claimed-by: SteveCastalk
merged-pr: https://github.com/NguyenKhacPhuc/undercurrent/pull/47
merged-at: 2026-06-19T06:30:07Z
wave: 0
estimate: 45m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/ios-mini-app-send-message
  - status/done
  - wave/0
---

# [kmp-common] A mini-app can ask the assistant on iOS, at parity with Android

**Lane:** kmp-common (shared handler + iOS host wiring)
**PRD section:** [[PRD]] Story 1
**API contract section:** n/a (no BE work)

## Why

HTML mini-apps reached iOS in the predecessor feature, but the
ask-the-assistant capability was deliberately left out — a mini-app that
asks the assistant works on Android and fails on iOS. This slice closes
that last parity gap: a mini-app on iOS can ask the assistant and get its
answer back.

## Acceptance criteria

- [ ] When a mini-app on iOS asks the assistant a question, the
      assistant's reply is returned to the mini-app.
- [ ] If the assistant isn't ready yet, the mini-app's request fails
      gracefully (a rejection it can handle) and the mini-app keeps
      working — no crash, no hang.
- [ ] The same mini-app asking the same question behaves the same on iOS
      and Android.
- [ ] Android's existing ask-the-assistant behavior is unchanged.
- [ ] The assistant exchange runs on the current conversation (lands in
      chat history) — same as Android; isolating it is out of scope.

## Blocked by

- nothing — independently grabbable. (Builds on `ios-mini-app-render`,
  already merged.)

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Read
> `undercurrent/CLAUDE.md` and open the actual files before editing.

- **The gap is one missing host binding.** Android's app DI wires a
  `MiniAppAssistantHandler` into the component registry; the iOS DI's
  registry block omits it, so `window.weft.sendMessage` resolves to a
  null handler and the bridge rejects the call. The substrate bridge +
  the rest of the iOS mini-app path already work (shipped in
  `ios-mini-app-render`).
- **Existing pattern to mirror / lift:** the Android handler is a private
  factory in `androidApp/.../di/AppModule.kt` (`miniAppAssistantHandler`)
  that runs a one-shot agent turn against the current agent and returns
  the last assistant reply. Expected move (see [[decisions]] D1/D2): lift
  it to shared code parameterized on the current agent (reachable via the
  shared `AgentSlot.agent` on both platforms — `AgentSlot` is already in
  iOS DI via `chatHostModule`), wire it into the iOS registry block in
  `IosKoinModule`, and refactor Android to use the shared factory.
- **No substrate change** — uses the existing
  `MiniAppAssistantHandler` contract from `weft-compose-defaults`.
- **Watch out for:** the kmp-common verify must compile **both** targets
  (`compileDebugKotlinAndroid` + `compileKotlinIosSimulatorArm64`) plus
  run the relevant module tests. If the shared handler lands in a module
  that doesn't yet have the agent types on its commonMain classpath,
  prefer a home that already does (e.g. the chat feature's commonMain)
  over adding a new dependency.

## Out of scope for this story

- Ephemeral-conversation isolation (the turn lands in chat history, same
  as Android) — separate tracked follow-up (html-mini-apps Q2 / k04).
- Any new mini-app capability or offerable-action change.
