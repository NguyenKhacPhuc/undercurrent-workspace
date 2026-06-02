---
type: open-questions
feature: weft-ios-parity
created: 2026-06-02
tags:
  - inception/open-questions
  - feature/weft-ios-parity
---

# Open questions

> [!question]
> Driver's parking lot. All items resolved with the driver — **this file is empty of open items**, so the Inception phase's open-questions gate is met.

## Open

### Q3 — iOS voice input (story 11) is blocked by a Kotlin/Native binding gap

- **Why it matters:** `Speech.recognize()` needs `AVAudioSession.setActive/setCategory/setMode`, which are **unresolved references** in Kotlin/Native 2.3.10 against the iOS 26.4 SDK (confirmed by compile probe in weft, 2026-06-02; undercurrent hit the identical wall and stubbed its `IosSpeechRepository`). Voice input cannot be implemented against the commonized `platform.AVFAudio` bindings as-is. Note the "lift undercurrent's impl" hint in [[11-ios-voice-input]] is void — that impl is itself a no-op stub.
- **[DRIVER GUESS]:** Fix via a **custom cinterop `.def`** in `:os-bridge` that explicitly binds `AVAudioSession` + `SFSpeechRecognizer`, OR wait for a Kotlin/Native release with refreshed iOS 26.4 bindings. Either is its own slice; [[11-ios-voice-input]] stays `ready` (deferred) until then. `say`/`stop` (TTS via `AVSpeechSynthesizer`) are unaffected and could ship separately.
- **[ASKED OF]:** iOS

## Resolved

### Q1 — Who owns the iOS permission usage-description strings? — 2026-06-02

- **Answer:** The **undercurrent host** owns adding the microphone + speech-recognition usage strings, as part of [[01-host-adopt-substrate-capabilities]]. The SDK documents which string each capability requires; it does not ship app-specific wording.
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D5, [[01-host-adopt-substrate-capabilities]] (already an acceptance criterion)

### Q2 — Debug-overlay parity depth on iOS? — 2026-06-02

- **Answer:** **Good enough to inspect a session** — the overlay opens and shows the current session's agent activity. Full feature-for-feature Android parity is a future follow-up if iOS developers ask. [[13-ios-debug-overlay]]'s acceptance criteria already reflect this.
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[13-ios-debug-overlay]] (acceptance criteria + out-of-scope)

### Q0 — How deep should iOS parity go? — 2026-06-02

- **Answer:** Foundation/mechanism only — turnkey setup + shared composition + sign-in launcher + 10 foundational capabilities + graceful fallbacks. ~20 capabilities deferred.
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D1, [[out-of-scope]]

### Q0b — Minimum iOS deployment target? — 2026-06-02

- **Answer:** iOS 15.
- **By:** Driver (SteveCastalk)
- **Promoted to:** [[decisions]] D2, [[PRD]] Constraints
