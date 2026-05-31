---
type: decisions
feature: sign-in-flow
created: 2026-05-31
tags:
  - inception/decisions
  - feature/sign-in-flow
---

# Decisions

> [!info]
> ADR-lite log. One entry per decision the mob made (or that the driver made and the mob ratified). The point is to prevent re-litigating the same argument in week 3.

## To ratify

### D1 — Sign-in is blocking on first launch (no "continue as guest") — 2026-05-31

- **Context:** Sign-in could be optional (skippable) or mandatory. Optional risks ~no one ever filling it in, which defeats the "have user info ready for future sync" purpose.
- **Options considered:** Blocking screen; skippable screen with "continue as guest"; optional Settings entry with no gate.
- **Decision:** Blocking. User cannot reach the home surface without completing sign-in.
- **Why:** The whole reason to add this now is to have user info on every install by the time BE-backed sync arrives. A skippable flow undermines that.
- **Consequences:** First-launch friction goes up. A user who refuses to provide an email cannot use the app. Mitigated partially by the local-only disclosure (see [[open-questions#Q4]]).

### D2 — No future-sync proofing in v1 (no reserved id, no repository swap-shape) — 2026-05-31

- **Context:** When BE eventually ships, we'll want a way to link the local profile to a backend account. Driver chose to skip all future-proofing — no reserved local id, no repository-shaped abstraction.
- **Options considered:** (a) local-only, no future-proofing; (b) reserve a stable `user.<uuid12>` locally; (c) build a `UserProfileRepository` interface that a future BE impl can drop into; (d) wire up real Apple/Google OAuth now.
- **Decision:** (a) local-only, no future-proofing.
- **Why:** YAGNI. The BE lane is dormant; speculating about its shape now is wasted work. When BE Inception starts, that team will spec the migration including any id generation it needs. Most BE onboarding flows can map by email anyway.
- **Consequences:** When BE arrives, we'll need a migration moment (probably "re-sign-in with Apple/Google, we'll match on email and merge your local data"). [[open-questions#Q5]] tracks the deferred work.

### D3 — Existing installs hit the same blocking screen on first open after upgrade — 2026-05-31

- **Context:** When this release ships, every existing install will have no profile. The flow has to do *something* for them. Options ranged from a soft banner to the same hard gate as fresh installs.
- **Options considered:** Same blocking screen; pre-fill heuristic + blocking; dismissible banner; silent (no nudge).
- **Decision:** Same blocking screen. "No profile present" is the only signal we check; we don't distinguish first-install from upgraded-install.
- **Why:** Simplest. One code path. Avoids the engagement risk of a dismissible banner (people would ignore it and the data never lands).
- **Consequences:** Existing users will be confronted with a name/email request on their next launch with no warning. We should write a short blurb at the top of the sign-in screen for this case OR ship a release note; mob to decide. See [[open-questions#Q4]] (same disclosure copy may cover this).

### D4 — Stay 100% commonMain; no Android-only or iOS-only stories — 2026-05-31

- **Context:** Per Undercurrent's KMP-default discipline, we ask each feature whether there's genuinely platform-specific impl. For sign-in: text fields, a button, a DataStore-Preferences write, a routing decision. All of that is commonMain.
- **Options considered:** All commonMain; iosMain uses Keychain for storage; androidMain wires up Credential Manager for prefill.
- **Decision:** All commonMain. DataStore-Preferences for the profile blob. No Keychain (display name + email are not secrets at the threat-model level we're operating at). No Credential Manager / iOS AutoFill prefill in v1.
- **Why:** Smallest possible footprint. The KMP shared shell is sufficient. We can always add a platform-specific story later if we discover a real need.
- **Consequences:** Issues live entirely under `issues/kmp-common/`. No `issues/android/` or `issues/ios/` dirs.

---

*(Future entries appended as the mob ratifies / overrides / adds.)*
