---
type: decisions
feature: ios-agent-bringup
created: 2026-06-02
tags:
  - inception/decisions
  - feature/ios-agent-bringup
---

# Decisions

> [!info]
> ADR-lite log. Driver decisions for the mob to ratify.

---

### D1 — MVP is a working text agent + integrations; Koog-bits and voice deferred — 2026-06-02

- **Context:** The iOS host is a ~10-stub shell. Full parity needs Koog (JVM-only) and voice (blocked). The driver wants a usable agent, not the whole long tail.
- **Options considered:** (a) text agent only (stories 1–6 of the scoping map); (b) text agent + OAuth integrations; (c) full epic incl. blocked stories.
- **Decision:** **(b)** — wire the real `WeftRuntime` + the shared data layer + secure keys + the streaming chat repo + app-lifecycle parity + **OAuth integrations** (now unblocked by the merged `IosOAuthClient`). Defer model-catalog + key-validation (Koog) and voice.
- **Why:** Delivers an actually-usable iOS agent with integrations. The deferred items are externally blocked, not just unfinished — building them now isn't possible.
- **Consequences:** iOS keeps the synthetic model catalog + no key-validation + hidden mic until those unblock. Tracked in [[out-of-scope]].

### D2 — Lift the Weft-backed layer to commonMain (de-dup), not iOS-only copies — 2026-06-02

- **Context:** The host's `Weft*Repository` impls + the chat agent host live in `androidMain` **only because `WeftRuntime` was Android-bound**. The substrate is now commonMain.
- **Options considered:** Lift the impls to commonMain (shared); or write parallel iOS-only copies.
- **Decision:** **Lift to commonMain.** Android and iOS run one implementation; iOS adds DI wiring only. This includes relaxing the host's "feature modules must not depend on Weft" rule for the chat feature, whose commonMain may now depend on the (KMP) substrate.
- **Why:** The whole point of the substrate KMP-ification is to stop duplicating the Weft-backed layer per platform. A copy that isn't shared drifts.
- **Consequences:** The lifts touch Android's source layout (androidMain → commonMain) — must verify zero Android behavior change. The chat feature gains a commonMain Weft dependency (previously forbidden); update `undercurrent/CLAUDE.md`'s KMP-discipline note.

### D3 — Model catalog, key validation (Koog), and voice are out of scope — 2026-06-02

- **Context:** Koog's LLM clients are JVM/Android-only; voice needs the AVAudioSession K/N binding (deferred in the substrate).
- **Decision:** Keep the iOS stubs for `ModelCatalogRepository` (synthetic), `KeyValidationRepository` (no-op), and `SpeechRepository` (hidden mic). Not in this feature.
- **Why:** Externally blocked — not buildable now.
- **Consequences:** iOS users paste a key without a validation ping and pick from a synthetic model list. Acceptable for MVP; documented in [[out-of-scope]].

### D4 — Construction branches from `undercurrent` origin/main — 2026-06-02

- **Context:** At Inception the `undercurrent` repo was on a dirty feature branch with uncommitted changes (`refactor/extract-auth-strings-to-resources`).
- **Decision:** All Construction branches off `undercurrent` `origin/main`, never the in-flight branch. Coordinate so the refactor branch and this feature don't collide.
- **Why:** Don't disturb someone else's uncommitted work; keep PRs reviewable against `main`.
- **Consequences:** Surfaced as [[open-questions]] Q1 for the mob to confirm the branch is settled / merged before the iOS DI stories land.
