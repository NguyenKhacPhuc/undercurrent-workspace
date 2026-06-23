---
type: decisions
feature: mini-app-exchange
created: 2026-06-19
tags:
  - inception/decisions
  - feature/mini-app-exchange
---

# Decisions — Mini-App Exchange

> [!info] Unilateral driver decisions to ratify in mob review. Each has a rationale so the mob can challenge the *why*, not just the *what*.

## D1 — Sign-in required to share; install is open

**Decision:** Creating a share link requires a signed-in account. Receiving, previewing, and installing require no account.

**Why:** Sharing needs a real author identity (for attribution and future abuse-tracing), and the user base already has accounts. Install must be friction-free or the viral loop dies — gating install behind sign-up would kill the single best growth path.

## D2 — Share links are permanent but revocable

**Decision:** Links do not expire. The author can "stop sharing" to revoke a link (it 404s afterward). Already-installed copies keep working.

**Why:** Expiry hurts long-lived useful tools and adds TTL plumbing for little v1 value. Revocation gives the author control over regret without expiry's downside. Installed copies are local, so revocation can't (and shouldn't) reach back to them.

## D3 — QR via OS camera + custom-scheme deep link; no in-app scanner

**Decision:** Sharing displays a QR encoding a custom-scheme deep link (`undercurrent://install/<shareId>`). Recipients scan with their phone's normal camera, which opens the app's preview. We do **not** build an in-app camera scanner in v1.

**Why:** An in-app scanner means camera permission + scanner UI on both platforms for a flow the OS already handles. Custom-scheme deep links also sidestep the need for app-association files (universal/app links), shrinking the per-platform slice to scheme registration + routing. The human-readable https link is shared as a secondary path; a polished web landing for "app not installed" is out of scope (see [[out-of-scope]]).

## D4 — Author shown as profile display name

**Decision:** The preview attributes the mini-app using the sharer's existing account display name, snapshotted at share time.

**Why:** Reuses identity we already have; no new profile surface (public handles, uniqueness) to design. Snapshotting means attribution is stable even if the account later renames.

## D5 — HTML size cap of 256 KB; saved state is not shared

**Decision:** A shared HTML document is capped at 256 KB. The mini-app's saved per-device state is **not** included in the bundle.

**Why:** Self-contained HTML mini-apps are small; 256 KB is generous headroom while bounding backend storage and QR/transfer cost. State is personal and per-device (a recipient starts fresh), so shipping it would be both a privacy leak and a correctness hazard. **[DRIVER GUESS on the exact number — see [[open-questions#Q1]].**

## D6 — A second install of the same share is a distinct local copy

**Decision:** Installing the same shared mini-app twice produces two independent local mini-apps, each with its own state and its own first-launch approval.

**Why:** Simplest predictable behavior; avoids a "merge vs replace" decision and de-dup matching. Edge case is rare. Revisit if it proves annoying.
