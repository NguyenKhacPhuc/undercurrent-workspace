---
type: feature-index
feature: mobile-auth-wiring
status: draft
created: 2026-06-01
tags:
  - inception/index
  - feature/mobile-auth-wiring
  - status/draft
---

# Mobile auth wiring — feature index

> [!info] **Status:** Draft / awaiting mob review
> Replaces the superseded [[../260531-1719-sign-in-flow/_index]] now that the BE has shipped. See [[decisions#D1]].

## Quick links

- PRD: [[PRD]]
- API contract: [[api-contract]] *(consumes the existing BE — 0 new endpoints, 0 TBDs)*
- Decisions: [[decisions]] *(D1–D7 to ratify)*
- Open questions: [[open-questions]] *(6 for mob)*
- Out of scope: [[out-of-scope]]
- Project-wide context: [[../../CONTEXT]]
- Backend Inception: [[../260531-1733-backend-bootstrap-auth/_index]]
- Superseded predecessor: [[../260531-1719-sign-in-flow/_index]]

---

## Parallel work plan

Issues are grouped into **waves** by dependency depth. All issues in a wave can be picked up simultaneously by different devs.

### 🟢 Wave 0 — start here (no blockers)

| Issue | Estimate | Lane |
|---|---|---|
| [[01-session-token-store-interface]] | 30m | kmp-common |
| [[04-be-auth-client]] | 60m | kmp-common |

### 🟡 Wave 1 — unlocked once Wave 0 lands

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[02-android-encrypted-session-token-store]] | 45m | android | `01` |
| [[03-ios-keychain-session-token-store]] | 45m | ios | `01` |
| [[05-first-launch-sign-in-screen]] | 90m | kmp-common | `01`, `04` |

### 🟠 Wave 2 — final slice

| Issue | Estimate | Lane | Blocked by |
|---|---|---|---|
| [[06-settings-account-and-sign-out]] | 60m | kmp-common | `05` |

---

> [!tip] **Parallelization sanity check**
> Wave 0 has 2 grabbable stories — interface + API client, independent of each other. Wave 1 fans to 3 (two platform token-store impls + the sign-in screen — all consume different ends of `01` so no file overlap expected). Wave 2 is a single small story; could be collapsed into 05 if a dev is doing both, but kept separate so a different person can take Settings while sign-in is in review.
>
> Total focused work ≈ 5–5.5 hours across 6 stories. With one dev: roughly 1 day. With two devs from Wave 1 onward (one on Android-side, one on KMP-screen): about half that.

---

## All issues

```
issues/
├── kmp-common/
│   ├── 01-session-token-store-interface.md
│   ├── 04-be-auth-client.md
│   ├── 05-first-launch-sign-in-screen.md
│   └── 06-settings-account-and-sign-out.md
├── android/
│   └── 02-android-encrypted-session-token-store.md
└── ios/
    └── 03-ios-keychain-session-token-store.md
```

The lane split reflects [[decisions#D2]] — secure token storage is genuinely platform-divergent (Keychain vs EncryptedSharedPreferences), so two stories live outside `kmp-common/`. Everything else is shared.

> [!note] **How to update this index as work lands**
> When an issue's status changes, update its frontmatter (`status: ready` → `in-progress` → `done`).

---

## Definition of done (whole feature)

- [ ] All 6 issues have status `done` in their frontmatter.
- [ ] [[open-questions]] has zero unresolved items.
- [ ] [[PRD]] has at least one success metric. *(currently: three — completion rate, 401-recoveries, sign-out usage.)*
- [x] [[api-contract]] has zero `TBD` markers. *(Consumes existing BE; nothing TBD.)*
- [ ] [[decisions]] reviewed by mob.
- [ ] End-to-end on a real Android device + a real iOS device: fresh install → sign-in screen → register → home → Settings shows account → Sign Out → back to sign-in screen → sign in with same creds → home.
- [ ] Stale-token recovery confirmed: while signed in, revoke the session server-side (via dashboard or a curl to the BE); next authed call returns 401 → app wipes token → routes to sign-in.
