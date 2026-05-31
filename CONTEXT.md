---
type: context
project: undercurrent
tags:
  - context
  - project/undercurrent
---

# CONTEXT

> [!note] **Project-wide.** This file lives at the workspace root, not inside any feature's Inception folder. It is the project's shared language and grows across features. Each feature's Inception run *appends* to it.

Shared language for Undercurrent. Every term here must earn its place by replacing a longer phrase the team would otherwise repeat. **If a term is only used once, delete it.**

## Domain terms

| Term | Meaning | Replaces |
|---|---|---|
| **Provider** | An LLM API endpoint configuration the user has a key for. One of: Anthropic, OpenAI, OpenRouter, DeepSeek. | "the LLM provider the user has configured" |
| **Persona** | A user-defined Voice + Role pair injected as a system-prompt fragment per turn. Voice is *how* the assistant talks, Role is *what hat* it wears. Either may be null. | "the persona the user picked — voice + role combined" |
| **Voice** | The "speaking style" slot of a Persona. One of `Default / Custom`. | "the persona's voice slot" |
| **Role** | The "hat" slot of a Persona. Built-in roles + user-created customs. Independent of Voice. | "the persona's role slot" |
| **Mini-app** | A user-saved prompt (name + emoji + trigger text) that fires a one-tap agent turn. May cache the last `ui_render` payload for instant render-on-tap. | "the user's saved feature / saved prompt" |
| **Conversation** | A persisted chat thread. Has an id, a title (first user message, ≤40 chars), a created-at and a last-message-at timestamp. | "the chat thread we're in" |
| **Integration** | An OAuth-gated external system the agent can call via MCP. Today: Linear. | "the connected external service" |
| **Agent** | A `WeftAgent` instance — a specific LLM + system prompt + tool set. Default agent vs Writer agent (system-prompt biases prose mode). | "the configured LLM call setup" |
| **Trace** | One agent turn's full audit record: LLM calls, tool invocations, outcomes, latency. Persisted via Weft's observability harness. | "the per-turn log of what the agent did" |
| **Memory** | A long-lived fact the agent has stored across conversations. Two scopes: `SESSION` (this conversation) and `PERMANENT`. | "the things the agent remembers between sessions" |
| **Tool** | A `WeftTool` subclass — a device or app action the agent can invoke (open_personas, set_theme_palette, show_location_on_map, …). | "an action the agent can take" |
| **Skill** | A documented capability the agent advertises (separate from Tool — Skills are about *what the agent knows it can do*; Tools are *how it does it*). | — |
| **Render tree** | A JSON `ui_render` payload the agent emits to declaratively describe a UI to draw. Rendered by Weft's `TreeRenderer`. | "the agent's declarative UI output" |
| **Substrate / SDK** | The Weft modules (`:runtime`, `:harness:*`, `:tools`, `:compose`, `:oauth`, `:mcp`, `:contracts`). Lives in the sibling `weft/` repo. | "the Weft SDK" |
| **Host app** | Undercurrent itself, as distinguished from the substrate. | "the app on top of the SDK" |
| **Repository** | A `*Repository` interface in `:core:domain`. Replaces the legacy "Gateway" name. Concrete impls per platform under `:core:domain/{androidMain,iosMain}`. | "the app's data port" |
| **Route** | The stateful entry-point Composable per feature (`<Name>Route`). `ScreenRouter` calls these; they own DI + intent dispatch. | "the screen's wrapper Composable" |
| **Screen** | The stateless Composable that renders state + emits callbacks. Previewable; takes no DI. | "the actual UI" |
| **User profile** | The locally-captured identity for the person using this install — currently `displayName` + `email`. Distinct from a server-backed *Account* (see below); they are NOT the same record. | "the local identity" |
| **Account** | The server-side identity record owned by the backend. Has a stable id (`acct.<uuid12>`), email (unique, lowercased), display name, and a `password_hash`. Introduced by the backend-bootstrap-auth Inception (`inception/260531-1733-backend-bootstrap-auth/`). | "the server-side user record" |
| **Session** | An opaque, server-stored token that authenticates a single device's requests against the BE. Issued by sign-up / sign-in; invalidated by sign-out; 30-day TTL. **Distinct from `Memory` scope `SESSION`** which means "this conversation". | "the BE auth token" |
| **Backend submodule** | The third workspace submodule at `backend/`, sibling to `weft/` and `undercurrent/`. Owns the Ktor app, Postgres schema, and the Railway deploy. | "the BE repo" |
| **Driver** | The one person running Inception with the AI partner between mob-review sessions. | — |
| **Mob** | The full team (Android + iOS) that reviews + answers `open-questions.md`. | — |

## Domain entities (data model)

The shape of the things Undercurrent reasons about. Not Repository API shapes — those live per feature in `api-contract.md` when relevant. These are the conceptual entities.

### Conversation

- **What it is:** A persisted chat thread between user and agent.
- **Identifier:** String id with prefix `conv.<uuid12>`.
- **Key fields:** `id`, `title` (first user message ≤40 chars), `createdAtMs`, `lastMessageAtMs`.
- **Lifecycle:** Created on first user turn of a new chat. Updated on every assistant reply. Deletable individually or via clear-all.
- **Relationships:** Owns many ChatMessages. Bound 1:N to active Provider + Agent at creation time (currently both global state).

### ChatMessage

- **What it is:** One user or assistant turn in a Conversation.
- **Identifier:** String id `msg.<uuid12>`.
- **Key fields:** `role` (`USER` / `ASSISTANT`), `content`, `agentName` (which agent produced it; nullable for user turns), `createdAtMs`.
- **Lifecycle:** Append-only within a Conversation. Deleted only when the parent Conversation is deleted.

### Persona

- **What it is:** A user-defined or built-in Voice or Role.
- **Identifier:** String id; built-ins use stable slugs (`voice.default`, `role.coach`, …); customs use `persona.<uuid12>`.
- **Key fields:** `id`, `name`, `tagline`, `systemPromptText`, `kind` (`Voice` / `Role` / `Custom`).
- **Lifecycle:** Built-ins ship with the app. Customs are user-created and deletable.

### MiniApp

- **What it is:** A user-saved trigger prompt.
- **Identifier:** String id `feature.<uuid12>` (prefix kept for back-compat with the original "Saved Features" naming).
- **Key fields:** `id`, `name`, `emoji`, `triggerPrompt`, `createdAtEpochMs`, `usageCount`, `lastRenderTreeJson?`, `lastRenderedAtEpochMs?`.
- **Lifecycle:** User-created. Usage count + last-render-tree update on each invoke. Deletable.

### ThemePrefs

- **What it is:** The user's theme selection.
- **Identifier:** Single global slot in DataStore-Preferences.
- **Key fields:** `palette` (`AppPalette` enum), `mode` (`ThemeMode` enum: `Light` / `Dark` / `System`).

### Integration

- **What it is:** An OAuth-gated MCP-backed external service.
- **Identifier:** Stable string id (`linear`, …).
- **Key fields:** `id`, OAuth config, MCP server URL.
- **Lifecycle:** Built-in catalog. User enables/disables → `IntegrationsRepository.setEnabled(...)` → process restart picks up the new MCP server set.

### AgentTrace

- **What it is:** Per-turn audit record produced by Weft's observability harness.
- **Identifier:** `id`, scoped per Conversation.
- **Key fields:** `conversationId`, `startEpochMs`, `endEpochMs`, `userMessage`, `finalAssistantMessage`, `status` (`RUNNING` / `COMPLETED` / `FAILED`), `feedback` (`NONE` / `THUMBS_UP` / `THUMBS_DOWN`).
- **Relationships:** Owns many LlmCallTrace + ToolCallTrace children.

### Account

- **What it is:** The server-side user record owned by the backend.
- **Identifier:** String id `acct.<uuid12>`.
- **Key fields:** `id`, `email` (unique, stored lowercased), `displayName` (≤ 40 chars), `passwordHash` (argon2id, never returned by any endpoint), `createdAtMs`.
- **Lifecycle:** Created by `POST /v1/auth/sign-up`. Read by `GET /v1/me`. No edit, no delete, no email-verification in v1 (see `inception/260531-1733-backend-bootstrap-auth/out-of-scope.md`).
- **Relationships:** Owns many Sessions. Lives in `backend/` only; the mobile clients never persist `passwordHash` and only mirror `id` + `displayName` + `email`.

### Session

- **What it is:** An opaque server-stored token authenticating a single device's requests.
- **Identifier:** Random high-entropy string presented as `Authorization: Bearer <token>`.
- **Key fields:** `token` (or its hash, depending on Construction's choice), `accountId`, `issuedAtMs`, `expiresAtMs` (issued+30d), `revokedAtMs?`.
- **Lifecycle:** Issued by sign-up + sign-in. Validated by DB lookup on every authenticated request. Revoked by sign-out (sets `revokedAtMs`). Multi-session per account is supported (signing in on a second device does not revoke the first).

### UserProfile

- **What it is:** The signed-in user's locally-stored identity.
- **Identifier:** Single global slot in DataStore-Preferences (one profile per install — no multi-account today).
- **Key fields:** `displayName` (non-empty, ≤40 chars), `email` (loose format check).
- **Lifecycle:** Created the first time the user completes the sign-in screen on a fresh install or upgrade-without-profile. Editable from Settings. Absence of a profile is what triggers the blocking sign-in screen on launch. Never deleted in v1.
- **Relationships:** None today. Future BE-backed sync will need to anchor a server account to this profile (migration TBD when that lands).

### MemoryEntry

- **What it is:** A long-lived fact the agent has stored.
- **Key fields:** `id`, `content`, `scope` (`SESSION` / `PERMANENT`), `createdAtEpochMs`.
- **Lifecycle:** Created by the `remember_fact` tool. Deletable individually or via clear-all. Surfaced to the model on every turn via Weft's memory harness.

## Glossary of process terms

| Term | Meaning |
|---|---|
| **Inception** | Phase-1 of AI-SDLC. Driver runs `/inception`; output is PRD + api-contract + per-lane issues with acceptance criteria. Done when `open-questions.md` is empty. |
| **Construction** | Phase-2. A dev picks a ready issue (no unresolved `Blocked by:`) and builds TDD-style. One PR per issue. |
| **Lane** | One of: `kmp-common`, `android`, `ios`, `substrate`, `backend`. Issues live in `inception/<feature>/issues/<lane>/`. As of the backend-bootstrap-auth Inception (2026-05-31), the `backend` lane is no longer dormant. |
| **Wave** | A dependency-grouped batch of issues. Wave 0 = "no blockers — start day 1". Higher waves wait for earlier waves to land. Documented in `_index.md`. |
| **Mob review** | Synchronous read-through of Inception drafts by the whole team. Resolves `open-questions.md`. |
| **Force-stop** | `adb shell am force-stop dev.weft.undercurrent` — needed when iterating on tools so `WeftRuntime` rebuilds its tool catalog. |
| **Backend dormant** | *(Historical — superseded 2026-05-31 by the backend-bootstrap-auth Inception.)* The state of the `backend` lane before any BE Inception ran: no BE, all features client-only, BE-related Inception steps skipped. Term retained for past-feature context only. |
