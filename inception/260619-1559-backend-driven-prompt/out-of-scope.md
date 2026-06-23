---
type: out-of-scope
feature: backend-driven-prompt
created: 2026-06-19
tags:
  - inception/out-of-scope
  - feature/backend-driven-prompt
---

# Out of scope — Backend-driven prompt v1

> [!info] Explicitly excluded. Several are natural follow-ons once the base pattern lands.

- **Other prompt material.** Personas (Voice/Role), the writer-agent bias fragment, creator-mode instructions, and tool descriptions stay compiled-in. They can adopt the same pattern later (see [[decisions#D1]]).
- **Targeting / experimentation.** Per-user prompts, cohorts, A/B tests, % staged rollouts, canary. v1 is one global config ([[decisions#D2]]).
- **Mid-session live refresh.** Changes apply on next launch, not within an open conversation ([[decisions#D3]]).
- **Admin / editor UI.** No in-app or web surface to edit, diff, preview, or roll back the prompt in v1. Updates are a protected operator action ([[decisions#D6]]).
- **Prompt version history / rollback.** Beyond the single current config and its revision marker, no stored history or one-click rollback.
- **A compiled-in emergency fallback prompt.** Explicitly excluded by [[decisions#D4]] — called out here so no one "helpfully" adds one back without re-opening that decision.
- **Localization / per-locale prompts.** One prompt text for all locales in v1.
- **Per-provider prompt variants.** No different prompt per LLM provider (Anthropic/OpenAI/…) in v1.
