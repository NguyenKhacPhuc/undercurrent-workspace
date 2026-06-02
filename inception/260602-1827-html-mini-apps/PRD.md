---
type: prd
feature: html-mini-apps
status: draft
created: 2026-06-02
tags:
  - inception/prd
  - feature/html-mini-apps
  - status/draft
---

# PRD: HTML mini-apps with an agent bridge

> [!info] **Status:** Draft / awaiting mob review · **Driver:** SteveCastalk · **Last updated:** 2026-06-02
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

Let the agent build **flexible HTML/JS mini-apps that can actually *do* things** — by giving the existing sandboxed HTML surface a narrow, permissioned JS↔agent bridge — as the Tier-B escape hatch alongside the native component palette.

## Problem

The agent renders mini-app UI from a **fixed native component palette** (147 components). That palette is excellent for structured, standard UI (receipts, charts, forms, cards) — but it has a hard ceiling: every novel shape (a custom calculator, a game, a bespoke dashboard, a conditional form, a one-off tool) needs an engineer to ship component #148. The agent can't invent UI the palette doesn't already have.

The substrate already ships an `HtmlComponent` (`runScripts = true`) that renders **arbitrary agent-authored HTML/CSS/JS** — so UI flexibility is solved. But it's explicitly *"sandboxed: no native bridge"*: the HTML can draw anything yet **can't call a tool, persist state, hit the network, or talk to the agent**. So it's flexible in look, inert in function.

This feature closes that gap: a permissioned `window.weft` bridge that lets agent-authored HTML mini-apps reach device/app actions, persist their own state, receive live updates, and hand requests to the assistant — turning the HTML surface from a static canvas into a real mini-app runtime, without an engineer per shape.

## Goals

Testable goals.

- [ ] An HTML mini-app's script can **call an app/device action** through the bridge and receive the result (e.g. fetch data, share, store a value).
- [ ] A mini-app **only reaches actions it declared and the user approved** — undeclared or denied calls are refused with a clear error.
- [ ] The **first time** a mini-app wants device/app actions, the user sees **what it's asking for** and approves or denies; the choice is remembered.
- [ ] A mini-app can **save and reload its own state** across opens.
- [ ] A mini-app **automatically adopts the app's look** (colors, typography) so it doesn't feel foreign.
- [ ] The app can **push live updates** into a running mini-app, and a mini-app gets **open/close lifecycle** signals (and several can run).
- [ ] A mini-app can **hand a request to the assistant** (fire an agent turn) and react to the result.
- [ ] A mini-app **can be saved as a one-tap mini-app** (an HTML doc + its declared actions + its state), cached for instant render-on-tap.
- [ ] A mini-app **can't reach the network or escape the sandbox** except through approved actions.

## Non-goals

- **Replacing the native component palette.** It stays the default for structured/standard UI; this is the flexible escape hatch, not a rewrite. See [[out-of-scope]].
- **Live native rendering inside a mini-app** (in-view camera viewfinder, a pannable native map, AR, 60fps native graphics) — those remain native components, not HTML. See [[out-of-scope]].
- **Backend / sync.** No server involved. Network from a mini-app goes through the existing host-allowlist policy, not a new BE.
- **An app store / sharing mini-apps between users.** Mini-apps are agent-authored and locally saved in v1.

## User stories

> [!note]
> Two "users": the **person using the app** (runs flexible mini-apps, controls what they can access) and the **agent** (authors the HTML/JS). Substrate stories describe the bridge's public-facing API surface in domain terms.

### Story 1 — A mini-app that does something

**As a** person using the app, **I want** a mini-app to actually perform actions (fetch, save, share), **so that** it's a real tool, not just a picture.

**Acceptance criteria:**
- [ ] A mini-app can perform an app/device action and show the result.
- [ ] If the action fails or isn't available, the mini-app gets a clear failure rather than hanging.

### Story 2 — I control what a mini-app can touch

**As a** person using the app, **I want** to see and approve what a mini-app wants to access before it can, **so that** an agent-built mini-app can't quietly do things I didn't allow.

**Acceptance criteria:**
- [ ] On first run, a mini-app that wants device/app actions shows me what it's asking for; I approve or deny.
- [ ] A mini-app can only use what I approved; anything else is refused with a clear message.
- [ ] My approval (or denial) is remembered for next time, and I can change it.

### Story 3 — Mini-apps that remember and feel native

**As a** person using the app, **I want** mini-apps to keep their state and match the app's look, **so that** they feel like part of the app.

**Acceptance criteria:**
- [ ] A mini-app reopens with the state it had (e.g. a tracker keeps its count).
- [ ] A mini-app uses the app's colors and typography by default.

### Story 4 — Live and interactive

**As a** person using the app, **I want** mini-apps to update live and work with the assistant, **so that** they can show changing data and do agent-powered things.

**Acceptance criteria:**
- [ ] A running mini-app can receive a live update from the app and reflect it.
- [ ] A mini-app can ask the assistant to do something and react to the answer.

### Story 5 — Save it as a mini-app

**As a** person using the app, **I want** to save a flexible mini-app for one-tap reuse, **so that** I can run my custom tools instantly.

**Acceptance criteria:**
- [ ] A flexible (HTML) mini-app can be saved and reopened with one tap, instantly.
- [ ] Its saved actions and state come back with it.

## Success metrics

- **The agent can ship a working mini-app the palette couldn't:** a demo HTML mini-app that calls ≥1 real action + persists state runs end-to-end on a device. Measured by manual e2e.
- **No silent capability:** every action a mini-app reaches passed through an explicit user approval; an un-approved call is provably refused. Measured by the scope-gate behavior + manual check.
- **Look parity:** a default-styled HTML mini-app reads as part of the app (theme tokens applied). Measured by manual review.
- **No Android regression / both targets compile** for the shared substrate + host work. Measured by CI.

## Constraints

- **Substrate feature** — the bridge lives in `weft/` (`:compose-defaults` `HtmlComponent` + the agent/tool layer); separate PRs from the host. The host (`undercurrent/`) adds the mini-app catalog + the approval UX.
- **Security is load-bearing** — agent-authored JS is semi-untrusted (prompt-injection risk). Per-mini-app declared scopes + user approval, network only via the existing allowlist policy, CSP + sandbox. See [[decisions]] D2.
- **Both platforms** — `WKWebView` (iOS) + Android `WebView`; the bridge is per-platform under one shared contract.
- No backend.

## Links

- API contract: `./api-contract.md` (no BE work)
- Context: `../../CONTEXT.md`
- Builds on: weft `:compose-defaults` `HtmlComponent` (already renders HTML/JS, sandboxed)
- Issues: `./issues/`
