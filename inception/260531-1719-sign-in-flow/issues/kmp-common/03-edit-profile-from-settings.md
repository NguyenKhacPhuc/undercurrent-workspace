---
type: issue
feature: sign-in-flow
lane: kmp-common
status: ready
wave: 1
estimate: 60m
blocked-by:
  - "[[01-user-profile-store]]"
tags:
  - inception/issue
  - lane/kmp-common
  - feature/sign-in-flow
  - status/ready
  - wave/1
---

# [kmp-common] Edit display name and email from Settings

**Lane:** kmp-common
**PRD section:** [[PRD#Story 2 — Editable profile from Settings]]
**API contract section:** n/a — no backend

## Why

Without an editor, a user who fat-fingers their email at sign-in is stuck with that typo forever (short of reinstalling the app). Allowing edit later is one of the four core PRD goals and lifts the cost of any single mistake at sign-in time.

## Acceptance criteria

- [ ] Settings shows the user's current display name and email somewhere clearly attributable to "your profile" / "you".
- [ ] From Settings, the user can open an editor for the profile (inline, modal, or sub-screen — Construction's choice).
- [ ] In the editor, the display-name and email fields are pre-filled with the current values.
- [ ] The user can change either field independently; saving updates only what changed.
- [ ] The same validation that gated the sign-in screen applies here: non-empty trimmed display name, loosely-valid email format. Failed validation shows an inline error and disables Save.
- [ ] Tapping Save persists the new values and dismisses the editor; Settings immediately reflects the updated values.
- [ ] Tapping Cancel (or otherwise dismissing without saving) discards the in-flight changes; Settings still shows the old values.
- [ ] After save, the new values survive an app restart.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes (Construction chooses the test shape).
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :androidApp:compileDebugKotlin`, `./gradlew :composeApp:compileKotlinIosSimulatorArm64`, and the affected module's `test` task.

## Blocked by

- [[01-user-profile-store]] — the editor reads + writes through the same store.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Construction reads
> the lane's CLAUDE.md + opens existing similar files and may diverge from
> any hint here without re-opening Inception.

- **Likely files affected:** the existing Settings feature module — find where it currently lists rows and append a profile section. The editor can live in the same module or as a small new module; Construction's call.
- **Existing pattern to mirror:** whichever Settings row currently lets the user mutate a setting and persist it (e.g. theme palette via `ThemePrefs`) — copy that flow. See [[../../../../CONTEXT#ThemePrefs]].
- **Watch out for:** if Construction factors out a shared `ProfileEditor` composable used by both this story and [[02-first-launch-sign-in-screen]], confirm both screens still pass their own AC after the refactor (small risk of cross-coupling regressions).
- **Watch out for:** Save should be idempotent — saving the unchanged values is a no-op (no write storm).

## Out of scope for this story

- Sign out / wipe profile from Settings. Not in v1; see [[../../open-questions#Q7]].
- Avatar upload from Settings. Not in v1; see [[../../out-of-scope]].
- Adding fields beyond display name + email.
