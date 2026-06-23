---
type: prd
feature: mini-app-exchange
status: draft
created: 2026-06-19
tags:
  - inception/prd
  - feature/mini-app-exchange
  - status/draft
---

# PRD: Mini-App Exchange (v1)

> [!info] **Status:** Draft / awaiting mob review · **Driver:** Phuc · **Last updated:** 2026-06-19
> See [[_index]] for the parallel-work plan and [[open-questions]] for unresolved items.

## One-line intent

Let a person share an HTML mini-app the assistant built for them, via a link and QR code, so anyone can preview it, see what it can do, and install it — with the recipient re-granting capabilities through the existing approval flow.

## Problem

Today a mini-app the assistant authors is trapped on the device that made it. There is no way to hand "the little water-tracker the assistant wrote me" to a friend, to another of your own devices, or to anyone else. That kills the single most distinctive thing Undercurrent does — *the assistant writes you a real, working, capability-bearing app on demand* — because it can never leave the room.

The reason no one ships "share the app the AI built" is that sharing executable UI is dangerous. Undercurrent already solved that half: an HTML mini-app declares the actions it wants, and the person approves them before anything runs. Sharing is the missing half — and because approval is re-run on the recipient's device, it can be added without weakening the safety model.

**Who has the problem:** every user who has had the assistant build them something useful and wanted to pass it on; and anyone trying to grow the product by word of mouth.

## Goals

Testable goals.

- [ ] A signed-in person can turn any of their HTML mini-apps into a shareable link and QR code.
- [ ] Anyone who opens that link or scans that QR sees a preview that names the mini-app, shows who shared it, and lists every capability it will be able to request.
- [ ] Installing from a preview adds the mini-app locally and, on first launch, runs the existing capability-approval prompt — nothing executes before the recipient approves.
- [ ] The set of capabilities a recipient can ever approve is clamped to what their own app offers — a shared mini-app can never gain a capability the recipient's app does not sanction.
- [ ] The person who shared a mini-app can stop sharing it; afterward the link no longer resolves, while anyone who already installed keeps their working copy.

## Non-goals

Promoted to [[out-of-scope]].

- Sharing native (trigger-prompt) mini-apps — v1 is HTML mini-apps only.
- A browsable catalog / discovery feed of other people's mini-apps.
- Remixing or editing an installed shared mini-app in chat.
- Reporting, moderation, ratings, or any trust-and-safety review pipeline.
- An in-app camera QR scanner (recipients use their phone's normal camera).
- A polished web landing page for link-opens when the app isn't installed.
- Versioning or update-push for an already-shared mini-app.
- The assistant initiating a share on its own (an agent-side "share this" tool).

## User stories

### Story 1 — Share a mini-app

**As a** person who has an HTML mini-app the assistant built, **I want** to turn it into a link and QR code, **so that** I can hand it to someone else or to my other device.

**Acceptance criteria:**
- [ ] Each HTML mini-app offers a "Share" action.
- [ ] Sharing requires being signed in; if not signed in, the person is prompted to sign in first.
- [ ] Sharing produces a link and a scannable QR code for that mini-app.
- [ ] The link can be sent through the system share sheet (messages, email, etc.).
- [ ] A mini-app that has been shared is visibly marked as shared.
- [ ] Native (trigger-prompt-only) mini-apps do not offer the Share action in v1.

### Story 2 — Preview a shared mini-app

**As a** person who received a share link or QR, **I want** to see what the mini-app is and what it can do before installing, **so that** I can decide whether to trust it.

**Acceptance criteria:**
- [ ] Opening a share link, or scanning the QR with a phone camera, brings up a preview inside the app.
- [ ] The preview shows the mini-app's name, icon, and a short description if the sharer provided one.
- [ ] The preview shows who shared it (the sharer's display name).
- [ ] The preview lists every capability the mini-app will be able to request once installed.
- [ ] The preview makes clear the author is not vetted by Undercurrent — the recipient is trusting them.
- [ ] If the link has been stopped/revoked or is unknown, the preview shows a clear "this mini-app is no longer available" message instead of installing anything.

### Story 3 — Install from a preview

**As a** person looking at a shared mini-app preview, **I want** to install it, **so that** it shows up among my mini-apps and I can use it.

**Acceptance criteria:**
- [ ] Installing from the preview adds the mini-app to the recipient's Mini Apps list.
- [ ] Installing does not require an account.
- [ ] The first time the installed mini-app is launched, the existing capability-approval prompt appears; nothing the mini-app declared runs before the recipient approves.
- [ ] Any capability the mini-app declared that the recipient's app does not offer is dropped from what can be approved — it is never silently granted.
- [ ] The installed mini-app records that it came from a shared link and who shared it.
- [ ] Installing the same shared mini-app twice does not silently corrupt or duplicate state in a confusing way (define and honor one behavior — e.g. a second install is a distinct copy).

### Story 4 — Stop sharing

**As a** person who shared a mini-app, **I want** to stop sharing it, **so that** the link stops working if I change my mind.

**Acceptance criteria:**
- [ ] A shared mini-app offers a "Stop sharing" action.
- [ ] After stopping, opening the old link shows the "no longer available" preview.
- [ ] People who already installed the mini-app keep their working copy after the share is stopped.
- [ ] A mini-app that is not currently shared does not offer "Stop sharing".

## Success metrics

How we know this worked, after launch. At least one.

- **Adoption of sharing:** ≥ 15% of users who own at least one HTML mini-app create at least one share link within 30 days of release.
- **Install conversion:** ≥ 30% of share-preview opens result in an install (measures whether the preview earns trust).
- **Reach:** number of installs that originate from a shared link per week (the viral-loop signal; target a positive and growing trend, no hard number for v1).

## Constraints

- **Backend is live** (Ktor + Postgres on Railway) and owns the shared-bundle store and link minting. Reuses the existing bearer-token auth and `BaseResponse<T>` envelope from the auth feature.
- **Safety invariant (non-negotiable):** sharing must never widen the recipient's capability envelope. The recipient's app already clamps declared capabilities to its offerable set; install must route through that same clamp and reset approval to un-consented.
- **HTML mini-apps are self-contained** (inline JS/CSS, no remote scripts) — this is what makes them portable; v1 relies on it and does not add a remote-asset story.
- **No in-app scanner** — install entry is via OS-level deep links opened from a link tap or a camera scan of the QR.

## Links

- API contract: `./api-contract.md`
- Decisions: `./decisions.md`
- Open questions: `./open-questions.md`
- Out of scope: `./out-of-scope.md`
- Project-wide context: `../../CONTEXT.md`
- Issues: `./issues/`
